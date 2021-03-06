# Proposed DSL specification

This document contains the specification for the DSL that will be exposed to the players.

## Grammar/format

Scripts written by the players follow a declarative style based on a `select -> filter -> action` approach.

### Select

First the player states the objects they want to give instructions to. This is the `select` phase. Proposed `select` keywords are:

  * `all`
  * `any`
  * `only`

The first two will boil down to the same selection but it helps to make the DSL more comfortable and natural to the user. The third one will only (hence the name) select one element from a collection.  

The selection keyword is followed by a collection of entities. This can be `workers`, `fighter` (singular form to be used with the `any` and the `only` keyword) and so on. Anything that the player actually owns. What we have now:

* `all workers`
* `any fighter`
* `only fighter`

### Filter

After we've grabbed all entities of a type, we probably want to filter them a bit further. The `filter` phase makes it possible to specify which entities we want to interact with and which ones we want to drop.

A filter is introduced by the `that` keyword. This goes well with both of the selection keywords. After the `that`, the user can specify a predicate that must be satisfied by the entities. 

Let's take a look at the things we've got now:

~~~~Groovy
all workers that {
  somePredicate
}
~~~~ 

The `filter` phase is optional of course.

### Action

The `action` phase contains the actual instructions to the filtered set of entities. We can imagine this as a `for-each` loop on the entities, therefore we can still make decisions based on the properties of the different entities.

The `do` keyword is used to start the `action` phase. This phase cannot be omitted.

### All together now

Now put all the phases together:

~~~~Groovy
all workers that {
  some predicate
} do {
  some action
}
~~~~

This format feels natural and provides lots of flexibility to the player. Let's take a look at the implementation side!

## Implementation

Even though this DSL might look a bit complex, it can be implemented very easily, thanks to the flexibility of Groovy. 

Keywords of the `select` phase can be provided through a custom `Script` class or `Binding`s (for more information on these: [docs](http://docs.groovy-lang.org/docs/latest/html/documentation/core-domain-specific-languages.html#_script_base_classes)). The entity collections should use `Binding`s too. Singular and plural forms (`worker`/`workers`) must be bound to the same collection. 

The `filter` and `action` phase can be implemented using [command chains](http://docs.groovy-lang.org/docs/latest/html/documentation/core-domain-specific-languages.html#_command_chains). Because Groovy lets us omit the parentheses for functions and their parameters, `that` can be a function that takes a closure, and the same goes for `do`. So far we've covered the base structure but not the predicates and the actions.

The closure parameter of the `filter` phase must return a boolean value that will be used when looping through the entities. It indicates whether the entity should be kept or thrown away. Only the entites kept will be used in the `action` phase. 

So far, so good! Except for one thing. The player does not have a reference for the entity processed by the filter. This would make the whole filtering process useless (and the action phase too, of course). But fear no! Groovy rescues us once again. From the [Groovy docs](http://groovy-lang.org/closures.html#implicit-it):

> When a closure does not explicitly define a parameter list (using `->`), a closure always defines an implicit parameter, named `it`.

The implicit `it` parameter suits our needs and is perfectly named too! However it is a pretty fragile construct to rely on. We should never trust implicit things, but make all stuff explicit. Luckily we can easily accomplish this using numerous different ways! For example we can set the `delegate` of the closure to be a map containing an `it` key with the value of the processed entity ([Groovy maps](http://groovy-lang.org/syntax.html#_maps), [@DelegatesTo](http://docs.groovy-lang.org/docs/latest/html/documentation/core-domain-specific-languages.html#section-delegatesto), [resolve strategy example](http://docs.groovy-lang.org/latest/html/api/groovy/lang/Closure.html#DELEGATE_ONLY)). We can do just the same for the `action` phase.

If an entity participates in multiple `action` phases then only the effects from the last one are preserved. Therefore different code organization can lead to different results. 

Of course this is not a complete guide on the implementation but just the basic idea. Lots of bindings should be provided to the scripts so the player can interact easily with game objects. This includes the map, the resources, the enemies and so on. We should give the players the data and they will forge the logic using the DSL.

## Examples

### Basic example with one S(F)A statement 

~~~~Groovy
all workers do {
  def resource = findClosestResourceTo it

  if ((distanceBetween it, resource) == 1) {
    it mine resource
  }

  if (it.resourceLevel == 100) {
    def base = findClosestBaseTo it

    it goto base 
  }
}
~~~~

Here `findClosestResourceTo`, `findClosestBaseTo` and `distanceBetween` are player-written functions.

### Two blocks with filter phase
We can split the one big statement from the previous snippet into separate ones:

~~~~Groovy
all workers do {
  def resource = findClosestResourceTo it

  if ((distanceBetween it, resource) == 1) {
    it mine resource
  }
}

any worker that {
  it.resourceLevel == 100
} do {
  def base = findClosestBaseTo it

  it goto base
}
~~~~ 

### Intelligent filter to avoid multiple actions for the same entity

Still we can do better! Those workers that have `resourceLevel` equal to 100 can't really mine, so we should not search for the closest resource to them.

~~~~Groovy
all workers that {
  canMine it
} do {
  def resource = findClosestResourceTo it

  if ((distanceBetween it, resource) == 1) {
    it mine resource
  }
}

any worker that {
  cannotMine it
} do {
  def base = findClosestBaseTo it

  it goto base
}
~~~~

Here we created two convenience functions to make the code more readable. This code snippet has better performance than the previous one. 

### Nested selection

If we keep in mind that the `SFA` statement is just a `for-each` loop with filtering built in then we can make nested loops in a pretty nice way!

~~~~Groovy
all fighters do {
  def currentFighter = it

  only enemy that {
    distanceBetween currentFighter, it == 1
  } do {
    currentFighter attack it
  }
}
~~~~

Here there are some things that should be inspected. First, we saved the `it` of the outer `select` so we can reference it in the inner `select`. Also we did not name it `fighter` because that way we'd have reassigned the globally available `fighter` set.

### Incorrect usage

Note that it's not possible to save local variables between the `filter` and the `action` phase, so this is not correct:

~~~~Groovy
all workers that {
  def resource = findClosestResourceTo it

  distanceBetween it, resource == 1
} do {
  it mine resource
}
~~~~

