# Game Specification

In this document the basic structure of the game module is covered. These are not the implementation details but just plans and guidelines on the future tasks.

## Entity System

A component-based entity system should be developed based on the following techniques:

  * [Component Pattern](http://gameprogrammingpatterns.com/component.html)
  * [Game Entities with Components](http://cowboyprogramming.com/2007/01/05/evolve-your-heirachy/)

There should be only one entity class, `GameObject` that can be specialized through different *components*. Components in most cases hold *global* state across entities, and entities only contain *local* fields and data. For example the movement speed is a global attribute but health is not. 

Example components include:

  * `PositionComponent`
  * `MovementComponent`
  * `HealthComponent`
  * `ScriptComponent`
  * `TypeComponent`
  * `InventoryComponent`
  * `ExperienceComponent`
  * `PhysicsComponent`

Some components may include local data fields. These components must be instantiated on a per entity basis. This should be taken into account when serializing entities. A perfect example would be the various `InventoryComponent`s.

### ExperienceComponent

The `ExperienceComponent` holds a map of skills mapped to experience points, somehow like this:

~~~~
    ATTACK -> 100
    DEFEND -> 30
~~~~ 

If a skill is not present in the map, it is automatically assumed to be **0**.

## Layered structure: DSL -> Entity System

Objects and problems addressed by the DSL are on a much higher abstraction level than the Entity System. Therefore a layered structured is created to provide a mapping from the DSL. 

The top layer is the DSL. DSL instructions and commands are high level actions for example: `go to the nearest resource`. Of course this cannot be executed immediately. A middle layer mapper should be provided that separates actions like this into smaller pieces. These pieces form the `GameCode` that can be executed on the lowest layer i.e. the Entity System. 

For non-user issued actions, like `structure building process`, a separate player, the `Environment` should be introduced.

The middle layer also adds various effects to the mapped DSL code. For example workers with higher XP should mine more resources in one turn. Basically it takes the entities that participate in an action and adds or substracts different values from their attributes based on their XP and some other effects (terrain, buffs, etc.).

In simpler words no computation takes place in the Entity System layer, only getters/setters called.

