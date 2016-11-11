# Map

The InfectNet game is played on a map which needs to fulfill the following conditions:
 - easy and fast generation
 - two basic types of tiles: caves and rocks
 - no isolated islands of caves
 - an evenly placement of resources and players
 - capable of hosting at least 20 players comfortably 

## Representation of the Map

For simlicity reasons, the map is represented by a 2D array containing `enum` types such as `cave` and `rock`.

At first 20 000 * 20 000 tiles are generated, to be able to host the maximum of 20 players and using a 1000*1000 tile per player standard. 

## Generating the Map

An algorithm for the basic cave-rock structured map generation can be found at [this site](https://gamedevelopment.tutsplus.com/tutorials/generate-random-cave-levels-using-cellular-automata--gamedev-9664)
It requires an implementation of a Cellular Automaton.

## Generating resources and InfectNests

The resources should be placed at random, but evenly throughout the map. Only tiles of the `cave` type should be considered to hold a resource. There can be blocks of resources forming a bigger querry. Ideally 20% of the cave should have resources on it, for the players to easily access.

As the game starts and a player enters it should recieve an InfectNest placed at some distance from the other player's InfectNests. 
This distance is an estimated 1000*1000 tile square around a player's base.

## Additional features

### Route searching
Some sort of route searching algorithm is required to support the player's commands, e.g. going to a resource:
```
    any own.worker execute {
    def resourcePosition = findResourceInRadius(current, 50) 
    // user-written

  current goto resourcePosition
}
```

As a first attempt an A* algorithm should be implemented, it should be satisfactory for this purpose.
It needs to be quite quick considering that it will be used often. It will find a path when it is first called by a `goto` command and every time the moving entity is stopped by any barrier, e.g. a buildig which was not there when the entity started to go along the original path.
### Softening the edges

This is only a design addition. For future implementation use [bitmasking](https://gamedevelopment.tutsplus.com/tutorials/how-to-use-tile-bitmasking-to-auto-tile-your-level-layouts--cms-25673).
