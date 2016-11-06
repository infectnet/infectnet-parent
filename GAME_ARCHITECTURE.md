# Game Architecture

Before reading this document, please take a look at the `GAME_SPEC` and the `DSL_SPEC` files, because they contain ideas used here when reasoning about the architecture.

## Entity System

Game entities should contain references to various *Components*. They must not contain any data fields, just fields for the *Components*.

To make the implementation easier, an `Entity` holds a reference to all of the possible *Components*. This makes easier to work with `Entity` objects. If an `Entity` does not make use of a `Component`, then it must use the null object of that particular *Component* type. This null object instance can be shared between the `Entity` objects.

Example of an `Entity` class:

~~~~Java
class Entity {
  private HealthComponent healthComponent;

  private PositionComponent positionComponent;

  private ExperienceComponent experienceComponent;

  /*
   *   Other components.
   */

  /*
   *  Constructor
   */

   /*
    * Getters and setters.
    */
}
~~~~

Sadly, the fields cannot be final, because that would result in a very-very long constructor. 

An example component:

~~~~Java
class HealthComponent {
  private int health;

  public HealthComponent(int health) {
    this.health = health;
  }

  public int getHealth() { return health; }

  public void setHealth(int health) { this.health = health; }
}
~~~~

Shareable component for entities without health:

~~~~Java
class NullHealthComponent extends HealthComponent {
  public NullHealthComponent() {
    super(0);
  }

  @Override
  public void setHealth(int health) { return; }
}
~~~~

The null object approach is much better than using the real `null` value because it can prevent lots of errors and does not include any overhead.

### TypeComponent

Maybe the most important *Component* is the `TypeComponent`. This component stores the type of the `Entity` as per the [Type Object pattern](http://gameprogrammingpatterns.com/type-object.html). Subclasses of the component must include a `static` factory method (or constructor) that creates an `Entity` of that particular type. This can be viewed as a type constructor. 

The `TypeComponent` may include a category (for example `Worker` or `Building`) along with a subcategory (`Worm`, `Factory`). This is yet to be implemented.

### OwnerComponent

Every `Entity` must have an owner exposed through the `OwnerComponent`. For non-user entities, the `Environment` should be specified as an owner.

### ViewComponent

Game tiles visible to a specific player are just the same as the ones visible to their entities. Therefore entities must have a view radius. This can be a simple integer. For example, if `viewRadius` is equal to `1` then

~~~~
00000
0xxx0
0xEx0
0xxx0
00000
~~~~  

the cells containing `x` are visible to the `Entity` (`E`).

## Entity Wrappers

The DSL code must not operate on the raw `Entity` objects and change their properties. Making changes on the entities must be done in a controlled way. This is why `Entity Wrapper` objects are introduced. Basically these are proxy objects that act like real entities in the DSL code.

An `Entity Wrapper` object is created on the fly based on the `TypeComponent` of the `Entity`. For example, a `FighterWrapper` can be created for *fighters*. 

Wrappers have getter methods (for example `getHealth`) that expose the properties of the entity to the DSL as well as the more interesting action methods. The player uses the action methods to give orders to its entities. As an example the `FighterWrapper` might contain an `attack` method.

It is not necessary to have a base `EntityWrapper` class. Why would a `ResourceWrapper` expose the `health` of a resource? That'd have a bad feeling. 

Specific methods can be added to Wrapper classes as `trait`s. These enable multiple inheritance as well as type checking. This is the fastest and the most elegant solution in my opinion. Illustration:

~~~~Groovy
class FighterWrapper implements GotoTrait {
  // methods, that all fighters expose
}

class WorkerWrapper implements GotoTrait {
  // methods, that all workers expose
}

trait GotoTrait {
  void goto(int x, int y) {
    // Can access the methods and fields of the implementing class.    
  }
} 
~~~~

Then in the DSL:

~~~~Groovy
worker goto x: 5, y: 10 

fighter goto x: 7, y: 8
~~~~ 

This is how one can add traits in a somewhat static way to a class. However it is also possible to add traits dynamically to **instances**. Also, this dynamic approach is about 2-4 times slower than the static one. Here's the way to do it:

~~~~Groovy
FighterWrapper wrapper = new FighterWrapper(); // note that we have to create an instance

def traitWrapper = wrapper.withTraits(GotoTrait); // returns a new instance with the trait attached

traitWrapper goto x: 7 y: 8
~~~~ 

The static way should be preferred, but we should not forget `withTraits` as its pretty powerful.

## RequestQueue, ActionQueue and Systems

Calling an action on a wrapper creates a `Request` object and places it into a `RequestQueue`. `Request` objects are just *immutable* data storage objects that have only one thing in common: a source `Entity` that initiated the request. They're called requests because they request a system to do some kind of action. An outline for the a `Request` class:

~~~~Java
abstract class Request {
  private final Entity source;

  protected Request(Entity source) {
    this.source = source;
  }

  public Entity getSource() {
    return source;
  }
}
~~~~

And then the `GotoRequest` and `GotoTrait`:

~~~~Java
final class GotoRequest extends Request {
  private final int x;

  private final int y;

  public GotoRequest(Entity source, int x, int y) {
    // set fields
  }

  // getters
}  
~~~~


~~~~Groovy
trait GotoTrait {
  void goto(int x, int y) {
    // Here queue is just a static method on RequestQueue for simplicity
    RequestQueue.queue(new GotoRequest(this.wrappedEntity, x, y));    
  }
}
~~~~

But what to the Systems do? Actually they register listeners on the `RequestQueue`. These listeners listen to different types of requests. When the player code finished executing, all requests in the queue get passed to the appropriate System using the registered listeners.

Systems are responsible for `Request -> Action` translation. Wait, what's an `Action`? 

### First things first: Why do we need Actions?

When we get `Requests` we can't just execute them blindly. We want all Entities to operate on the same state of our world. So first we have to collect what everyone's wanna do, then we can do it. In simpler words: First we call all the getters and then we only call setters. This way we take the state of the world, make some computations and decisions based on that state. This is done for every request. After that, we no longer depend on the previous state of the world, so we can make any alteration we want. That simple.

Now let's talk about the two different approaches to the `Action`, 

### Actions - with logic

Here it's something that contains actual executable code. For example we can create a `GotoAction` just like this:

~~~~Java
class GotoAction extends Action {
  private final Entity target;

  private final int x;

  private final int y;

  public GotoAction(...) {
    ...
  }

  /*
   *  Defined in the abstract Action class.
   *
   *  Why do we pass the World? Side effects actually.
   */
  @Override
  public void execute(World world) {
    if (!world.getMap().getTileAt(x, y).isOccupied())
      target.getPositionComponent().setX(x);

      target.getPositionComponent().setY(y);

      world.getMap().getTileAt(x, y).setEntity(target);
    }
  }
}
~~~~  

Note that we must pass the `World` to the `Action` so it can make changes to it. If `Action`s contain code then the execution process is pretty simple:

~~~~Java
for (Action action : ActionQueue.actions()) {
  action.execute();
}  
~~~~

However it must be clear that using this approach, actions must get their depencencies injected into them, for example the `World`. Also, actions are separated in the sense, that they don't have memory. They have no knowledge of each other.

### Actions - only data

Using this approach, actions only consist of precomputed data that's ready to be set on the entities. The `GotoAction` is completely same as the `GotoRequest`. But someone's got to execute the actions... Yes, you're right, listeners again! Systems can listen to actions too. They register listeners on the `ActionQueue` and wait for actions. 

If we only store data in actions, they are harder to execute, but we also concentrated the logic in the Systems. That's why this approach might be better than the one before.

It's also easier to add reactions. For example, after an `AttackAction`, the attacked `Entity` might be dead. So in the listener for the `AttackAction` we create and queue a `ReaperAction` that wipes out the dead `Entity`. On the other hand we must be **very** careful with this. From [Game Programming Patterns](http://gameprogrammingpatterns.com/event-queue.html#you-can-get-stuck-in-feedback-loops):

> All event and message systems have to worry about cycles:
>
> 1. A sends an event.
> 1. B receives it and responds by sending an event.
> 1. That event happens to be one that A cares about, so it receives it. In response, it sends an event…
> 1. Go to 2.
>
> ... A common rule to avoid this is to avoid sending events from within code that’s handling one.

However, if we stay away from using this technique very frequently, it can be used efficiently, as I don't really see another solution on how these action-reactions things can be implemented.

## Sidenote - selectors

DSL selectors should be unmodifiable collections and must be namespaced, so they are easier to use. In other words we no longer have `fighters` or `enemy`. We have:

~~~~Groovy
own.fighters

enemy.workers

environment.resources
~~~~

In my opinion this is makes more expressive and useful the DSL. Selectors doesn't have to implement any interface but just return an unmodifiable collection, the player can work with.