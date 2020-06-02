# Reconquering Empyre

## About
This is a (shoddy written) python3 port of [VMS Empire](http://www.catb.org/~esr/vms-empire/) using ncurses. This is my 1st big project (the other one is relatively small), so there is going to be a lot of SNAFU and stuff.

## Requirements
* Python 3
* noise


### Startup parameters
* `-f [1,2,3,4]` Determine the Fog of War
  * 1: See the full map and all the units.
  * 2: See the full map but only friendly units
  * 3 (default): Map is unexplored but once discovered all units will remain visible.
  * 4: Map is unexplored and the only units visible are units within visual range of friendly units
* `-m xxx/yyy` Map size for horizontal and vertical axis from 60-999. Defaults are 100x50
* `-d delay` This option controls the length of time the computer will delay after printing informational messages at the top of the screen. delay is specified in seconds. The default value is 2 which allows the user two seconds to read a message.


### Keys
Depending on the context the following keys issue the following commands. There are the following contexts:
* `global` Commands that work no matter the context
* `city` Commands that work with cities
* `unit` Commands that work with units

#### Global
* `l` Display the logs
* `g` Give the AI 1 turn
* `n` Give the AI x turns
* `q` Save and Quit

#### City
* `p` Pause, do nothing
* `b` Build a unit
  * `a` army
  * `f` fighter
  * `p` patrol boat
  * `d` destroyer
  * `s` submarine
  * `t` troop transport
  * `c` aircraft carrier
  * `b` battleship
  * `z` satellite

#### Unit
* `Num 12346789` Direction to move/attack/load a unit
* `u` Unload a unit from Transport or Carrier
* `s` Turn unit into a sentry for 10 rounds
* `w` Set a waypoint for units to move to


## Notes

### Combat

```    while (att_obj->hits > 0 && def_obj->hits > 0) {
        if (irand (2) == 0) /* defender hits? */
            att_obj->hits -= piece_attr[def_obj->type].strength;
        else def_obj->hits -= piece_attr[att_obj->type].strength;
    }
```

|Piece|You|Enemy|Moves|Hits|Str|Cost|
| --- | ---| ---| ---| ---| ---| ---|
|Army|A|a|1|1|1|5(6)|
|Fighter|F|f|8|1|1|10(12)|
|Patrol Boat|P|p|4|1|1|15(18)|
|Destroyer|D|d|2|3|1|20(24)|
|Submarine|S|s|2|2|3|20(24)|
|Troop Transport|T|t|2|1|1|30(36)|
|Aircraft Carrier|C|c|2|8|1|30(36)|
|Battleship|B|b|2|10|2|40(48)|
|Satellite|Z|z|10|--|--|50(60)|

Other details: http://www.catb.org/~esr/vms-empire/vms-empire.html


```
typedef struct piece_attr {
        char sname; /* eg 'C' */
        char name[20]; /* eg "aircraft carrier" */
        char nickname[20]; /* eg "carrier" */
        char article[20]; /* eg "an aircraft carrier" */
        char plural[20]; /* eg "aircraft carriers" */
        char terrain[4]; /* terrain piece can pass over eg "." */
        uchar build_time; /* time needed to build unit */
        uchar strength; /* attack strength */
        uchar max_hits; /* number of hits when completely repaired */
        uchar speed; /* number of squares moved per turn */
        uchar capacity; /* max objects that can be held */
        long range; /* range of piece */
} piece_attr_t;
```

### Gamestate format
The `gamestate` is a variable that tells the program the state of the game. It is updated constantly and through the [Pickle](https://docs.python.org/3/library/pickle.html) to store the game to a file. I should make it save as a JSON at some point in case people want to create programs other than python to play from savegames.

#### gamestate['metadata']
This contains metadata about the game, what parameters was used to feed the Perlin Noise generator, size of the map, rounds etc
* width - width of the map
* height - height of the map
* coast_percentage - the percentage of cities that are coastal cities
* scale
* base
* octaves
* persistence
* lucnarity
* xoffset
* yoffset
* turn - who's turn it is, either AI or player
* round - int

#### gamestate['terrain']
This contains the blueprint of the terrain. It is a two dimensional array with either the one of the following values:
* `l` Land
* `w` Water
* `c` City

#### gamestate['coast']
A list of positions that are costs. Used for city placement. May be useful later on.

#### gamestate['cities']
List of cities, per city it stores the following:
* `location` x/y coordinate
* `side` Either `player` or `ai`
* `order` What its building or doing
* `stage` How far it is doing this

#### gamestate['units']
List of units, per unit it stores the following:
* `type` The unit-type, army, fighter etc
* `side` Either `player` or `ai`
* `location` x/y coordinate
* `order` Standing order for the units (`sentry`, `waypoint`)
* `stage` a counter or x/y coordinate
* `moves` How many moves are left
* `hits` How many hits are left

#### gamestate['fog']
A map with the Fog of War. Not sure about what has been explored or what is still fog.
