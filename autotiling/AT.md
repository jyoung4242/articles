---
slug: Autotiling Technique
title: Autotiling Technique
authors: [justin]
tags: [gamedev tilemap tiling autotiling tooling]
---

<img src="image-1.png" alt="TitleImage" style="width:400px;"/>

As a game developer, if the thought of hand crafting a level does not appeal to you, then you may consider looking into procedural
generation for your next project. Even using procedural generation, however, you still need to be able to turn your generated map
arrays into a tilemap with clean, contiguous walls, and sprites that match up cleanly, as if it was drawn by hand. This is where a
technique called auto-tiling can come into play to help determine which tiles should be drawn in which locations on your tilemap.

In this article, I will explain the concept of auto-tiling, Wang Tiles, [binary](https://en.wikipedia.org/wiki/Binary) and
[bitmasks](<https://en.wikipedia.org/wiki/Mask_(computing)>), and then walk through the process and algorithms associated with using
this tool in a project.

## What is auto-tiling

Auto-tiling converts a [matrix](https://processing.org/tutorials/2darray) or [array](https://en.wikipedia.org/wiki/Array) of
information about a map and assigns the corresponding tile texture to each tile in a manner that makes sense visually for the tilemap
level. This uses a tile's position, relative to its neighbor tiles to determine which tile sprite should be used. Today we will focus
on bitmask encoding neighbor data, although there are other techniques that can be used to accomplish this.

One can get exposed to auto-tiling in different implementations. If you're using a game engine like [Unity](https://unity.com/) or
[Godot](https://godotengine.org/), there are features automatically built into those packages to enabling auto-tiling as you draw and
create your levels. Also, there are software tools like [Tiled](https://www.mapeditor.org/), [LDTK](https://ldtk.io/), and
[Sprite Fusion](https://www.spritefusion.com/), that are a little more tilemap specific and give you native tools for auto-tiling.

Auto-tiling has provided the most benefit when we think about how we can pivot from tilemap matrices or flat indexes representing the
state of a tilemap, to a rendered map on the screen. Let us say you have a tilemap in the form of a 2d matrix with 1's and 0's in it
representing the 'walkable' state of a tile. Let us assign a tile as a floor (0) piece or a wall (1) piece. Now, one can simply use two
different tiles, for example:

a grass tile <img src="image-4.png" alt="grass tile" style="width:30px;"/> and a dirt path tile
<img src="image-3.png" alt="path tile" style="width:30px;"/>

We could take a tilemap matrix like this:

<img src="image-5.png" alt="tilemap matrix" style="width:300px;"/>

and use these two tiles to assign the grass to the 1's and the 0's to the path tile. It would look like this:

<img src="image-7.png" alt="rudementary tilemap" style="width:300px;"/>

This is technically a tile-map which has been auto-tiled, but we can do a little better.

## What are Wang tiles?

Wang tiles do not belong or associate with game development or tile-sets specifically, but come from mathematics. So, why are we
talking about them? The purpose of the Wang tiles within the scope of game development is to have a series of tile edges that create
matching patterns to other tiles. We control which tiles are used by assigning a unique set of bitmasks to each tile that allows us
reference them later.

Wang tiles themselves are a class of system which can be modeled visually by square tiles with a color on each side. The tiles can be
copied and arranged side by side with matching edges to form a pattern. Wang tile-sets or Wang 'Blob' tiles are named after Hao Wang, a
mathematician in the 1960's who theorized that a finite set of tiles, whose sides matched up with other tiles, would ultimately form a
repeating or periodic pattern. This was later to be proven false by one of his students. This is a massive oversimplification of Wang's
work, for more information on the backstory of Wang tiles you can be read here: [Wang Tiles](https://en.wikipedia.org/wiki/Wang_tile).

This concept of matching tile edges to a pattern can be used for a game's tilemap. One way we can implement Wang tiles in game
development is to create levels from the tiles. We start with a tile-set that represents all the possible edge outcomes for any tile.

These tile assets can be found here: [Wang Tile Set](https://opengameart.org/content/wang-%E2%80%98blob%E2%80%99-tileset)

<img src="wang.png" alt="wang blob" style="width:300px;"/>

The numbers on each tile represents the bitmask value for that particular permutation of tile design. We then can see how you can swap
these tiles for a separate texture below. In the image above, there are a couple duplicate tile configurations, and they are shown in
white font.

<img src="wang texture.png" alt="wang texture" style="width:300px;"/>

The magic of Wang tiles is that it can be extended out and create unique patterns that visually work. For example:

<img src="wang example.png" alt="wang texture" style="width:400px;"/>

A bitmask is a binary representation of some pattern. In the scope of this conversation, we will use a bitmask to represent the 8
neighbors tiles of an given tile on a tilemap.

## What is a bitmask?

A bitmask is a binary representation of some pattern. In the scope of this conversation, we will use a bitmask to represent the 8
neighbors tiles of an given tile on a tilemap.

### Quick crash course on Binary

So our normal counting format is designed as base-10. This means that each digit in our number system represents digits 0-9 (10
digits), and the value of each place value increases in power of base 10.

So in the number '42', the 2 represents - (2 \* 10<sup>0</sup>) which is added to the 4 in the 'tens' place, which is (4 \*
10<sup>1</sup>), which equals 42.

```
(2 * 1) + (4 * 10) = 42
```

T This in binary looks different, as binary is base-2, which means that each digit position has digits 0 and 1, (2 digits). This is the
counting system and 'language' of computers and processors.

Quickly, let's re-assess the previous example of '42'. 42 in binary is 101010. Let's break this down in similar fashion.

Starting from the right placeholder and working our way left... The 0 in the right most digit represents 0 \* 2<sup>0</sup>. The next
digit represents 1 \* 2<sup>1</sup>... and on for each digit and the exponent increases each placeholder.

```
   0         1         0         1         0          1
_________________________________________________________________
(0 * 1) + (1 * 2) + (0 * 4) + (1 * 8) + (0 * 16) + (1 * 32) = 42

2 + 8 + 32 = 42
```

### Bits, Bytes, and Bitmasks

That is how information in computers is encoded. We can use this 'encoding' scheme to easily represent binary information, like 'on' or
'off', or in this discussion, walkable tile or not walkable. This is why in the tile-set matrix example above, we can flag non-walkable
tiles as '1', and walkable tiles as '0'. This is now binary encoded.

A bit is one of these placeholders, or one digit. 8 of this bits together is a byte. Computers and processors, at a minimum, read at
least a byte at a time.

We can use this binary encoding for the auto-tiling by representing the state of each of a tile's neighbors into 8 bits, one for each
neighbor. This means that the condition and status of each neighbor for a tile can be encoded into one byte of data (8 bits) and CAN be
represented with a decimal value, see my earlier explanation about how the number 42 is represented in binary.

So the whole point of this section is to get to this example: we are going to encode the neighbor's data for an example tile.

### Quick Demonstration

<img src="image-8.png" alt="bitmask example" style="width:250px;"/>

Now the tile we are assigning the bitmask to is the green, center tile. This tile has 8 neighbors. If I start reading the 1's and 0's
from the top left and reading right, then down, I can get the value: 101 (top row) - 01 (middle row) - 101 (bottom row). Remember to
skip the green tile.

<img src="image-15.png" alt="bitmask example" style="width:250px;"/>

All together, this is 10101101, which can be stored as a binary value, which can be converted to a decimal value: 173. Remember to
start at the rightmost bit when converting.

```
   1         0         1         1         0          1          0          1
__________________________________________________________________________________
(1 * 1) + (0 * 2) + (1 * 4) + (1 * 8) + (0 * 16) + (1 * 32) + (0 * 64) + (1 * 128)

1 + 4 + 8 + 32 + 128 = 173
```

Now we can use that decimal value of 173 to represent the neighbor pattern for that tile. Every tile in a tilemap, can be encoded with
their 'neighbors' bitmasks.

As you saw earlier, the wang tiles had bitmask values assigned to them. This is how we know which tile to substitute for each bitmask.

## The Process

We have already covered the hard part. In this section we are pulling it all together in a walkthrough of the overall high level
process.

Here are the steps we are covering:

Find or create a tile-set spritesheet that you would like to use Create your tilemap data, however you like. Loop through each index of
tile, and evaluate the neighbor tiles, and assign bitmask Map the bitmap values to the 'appropriate' tile in your tile-set (this is the
long/boring part IMO) Iterate over each tile and assign the correct image that matches the bitmask value Draw your tilemap in your game
Creating a tile-set Here is an example of a tile-set that I drew for the demo project.

<img src="image-10.png" alt="example tileset" style="width:300px;"/>

These 47 tiles represent all the different 'wall' formations that would be required. I kept my floor tiles separate in a different file
so that it is easier to swap out. The floor is drawn as a separate tile underneath the wall. Each tile represented in the grid is
designed to match up with a specific group of neighbor patterns. Let's take the top-left tile:

<img src="image-11.png" alt="topleft tile" style="width:150px;"/>

This tile is intended to be mapped to a tile where there are walled neighbors on the right, below, and bottom right of the tile in
question. There maybe a few neighbor combinations ultimately that may be mapped to this tile, in my project I found 7 combinations that
this tile configuration would be mapped to.

If you look through each tile you can see how it 'matches' up with another mating tile or tiles in the map. For my implementation, I
spent time testing out each configuration visually to see which tile different bitmasks needed to be mapped to.

### Create your tilemap data

Now we will use either a 2d matrix or a flat array in your codebase, with each index representing a tile. I use a flat array, with a
tilemap width and height parameter. It is simply preference.

You can manually set these values in your array, or you can use a procedural generation algorithm to determine what your wall and floor
tiles. I can recommend my [Cellular Automata](https://dev.to/excaliburjs/cellular-automata-171f) aarticle that I wrote earlier if you
are interested in generating the tilemap procedurally. When this is completed, you'll have a data set that will look something like
this.

<img src="image-5.png" alt="Tilemap" style="width:250px; "/>

### Loop through tilemap and assign bitmasks

For each index of your array, you will need to capture all the states of the neighbor tiles for each tile, and record that value on
each tile. I would refer to the previous section regarding how to calculate the bitmasks.

```ts
    // This loops through each tile in the tilemap
    private createTileMapBitmasks(map: TileMap): number[] {
        // create the array of bitmasks, the indexes of this array will match up to the index
        // of the tilemap
        let bitmask: number[] = new Array(map.columns * map.rows).fill(0);
        let tileIndex = 0;

        // for each tile in the map, add the bitmask to the array
        for (let tile of map.tiles) {
            bitmask[tileIndex] = this._getBitmask(map, tileIndex, 1);
            tileIndex++;
        }

        return bitmask;
  }

  // setting up neighbor offsets indexes /
    const neighborOffsets = [
        [1, 1],
        [0, 1],
        [-1, 1],
        [1, 0],
        [-1, 0],
        [1, -1],
        [0, -1],
        [-1, -1],
    ];

  // iterate through each neighbor tile and get the bitmask based on if the tile is solid
    private _getBitmask(map: TileMap, index: number, outofbound: number): number {
        let bitmask = 0;

        // find the coordinates of current tile
        const width = map.columns;
        const height = map.rows;
        let y = Math.floor(index / width);
        let x = index % width;

        // loop through each neighbor offset, and 'collect' their state
        for (let i = 0; i < neighborOffsets.length; i++) {
            const [dx, dy] = neighborOffsets[i];
            const nx = x + dx;
            const ny = y + dy;
            //convert back to index
            const altIndex = nx + ny * width;

            // check if the neighbor tile is out of bounds, else if tile is a wall ('solid') shift in the bitmask
            if (ny < 0 || ny >= height || nx < 0 || nx >= width) bitmask |= outofbound << i;
            else if (map.tiles[altIndex].data.get("solid") === true) bitmask |= 1 << i;
        }

        return bitmask;
    }
```

### Map bitmask values to each tile sprite in spritesheet

Here is the monotonous part. For a byte, or an 8-bit word, the amount of permutations of tile patterns is 256. That's a lot of
mappings. Now I did mine the hard way, manually, one by one. But there may be easier ways to do this. I use Typescript, so I will share
a bit of what my mappings look like. Each number key in the object is the bitmask value, and its mapped to a coordinate array [x, y]
for my spritesheet that I shared earlier in the article. Now, I could have put them in order, but that does not really serve any
benefit.

```ts
export const tilebitmask: Record<number, Array<number>> = {
  0: [3, 3],
  1: [3, 3],
  4: [3, 3],
  128: [3, 3],
  32: [3, 3],
  11: [0, 0],
  175: [0, 0],
  15: [0, 0],
  47: [0, 0],
  207: [0, 5],
  203: [0, 5],
  124: [3, 5],
  43: [0, 0],
  ...
```

### Iterate over the tiles and assign tile sprite

The last two steps we'll do together. Now we simply need to iterate over our tilemap, assign the appropriate sprite tiles. I'm using
Excalibur.js for my game engine, and the code is in Typescript, but you can use whichever tool you would prefer.

```ts
draw(): TileMap {
    // call the method that loops through and configures all the bitmasks
    let bitmask = this.createTileMapBitmasks(this.map);
    let tileindex = 0;

    for (const tile of this.map.tiles) {
      tile.clearGraphics();

      // if the tile is solid, draw the base tile first, THEN the foreground tile
      if (tile.data.get("solid") === true) {
        // add floor tile
        tile.addGraphic(this.baseTile);

        // using the tile's index grab the bitmask value
        let thisTileBitmask = bitmask[tileindex];

        // this is the magic... grab the coordinates of the tile sprite from tilebitmask, and provide that to Excalibur
        let sprite: Sprite;
        sprite = this.spriteSheet.getSprite(tilebitmask[thisTileBitmask][0], tilebitmask[thisTileBitmask][1]);

        //add the wall sprite to the tile
        tile.addGraphic(sprite);

      } else {
        // if the tile is not solid, just draw the base tile
        tile.addGraphic(this.baseTile);
      }
      tileindex++;
    }
    return this.map;
  }
```

## Demo Application

<img src="image-1.png" alt="TitleImage" style="width:500px;"/>

[Link to Demo](https://mookie4242.itch.io/autotiling-demonstration)

[Link to Repo](https://github.com/jyoung4242/CA-itchdemo)

In this demo application, I'm using Excalibur.js engine to show how auto-tiling can work, and its benefits in game development. The
user can click on the tilemap to draw walkable paths onto the canvas. As the walkable paths are drawn, the auto-tiling algorithm will
automatically place the correct tile in its position based on the neighbor tile's walkable status.

There are some controls at the top of this app, a button to reset the tilemap settings back to not walkable, so one can start over.
Also, two drop downs that let the user swap out tile-sets for different styles. This shows the benefits of having standardized Wang
tiles for your tile-sets. For example, in this demo, we have three Wang tile-sets. When you swap them out, it can automatically draw
them correctly into your tilemap.

Grass

<img src="image-16.png" alt="Snow Tileset" style="width:300px;"/>

Snow

<img src="image-12.png" alt="Snow Tileset" style="width:300px;"/>

and Rock

<img src="image-14.png" alt="Rock Tileset" style="width:300px;"/>

# Why Excalibur

<img src="ex.png" alt="ExcaliburJS" style="width:750px;"/>

Small Plug...

[ExcaliburJS](https://excaliburjs.com/) is a friendly, TypeScript 2D game engine that can produce games for the web. It is free and
open source (FOSS), well documented, and has a growing, healthy community of gamedevs working with it and supporting each other. There
is a great discord channel for it [JOIN HERE](https://discord.gg/ScX52wD4eM), for questions and inquiries. Check it out!!!

# Conclusions

TThat was quite a bit, no? We covered the concept of Autotiling as a tool you can use in game developement. We discussed the benefits
of Wang tiles for your projects and that they allow for the auto selection of the correct tile sprites to use based off of bitmask
assignments. We dug into bitmask and base-2 binary encoding a little bit just to show how we were encoding the neighbor tile
information into a decimal value so we could map the tile sprites appropriately. We finished this portion by doing an example tile
encoding of neighbors to demonstrate the process.

We went step by step throught he process of autotiling, looking at tilesets, looking at code snippets, and finishing at the demo
application on itch. I hope you enjoyed this take on autotiling, as mentioned above, this is NOT the only way to do this, there are
other ways of accomplishing the same effect. You also can tweak this to your own liking, for instance, you can introduce varying tiles
so you can use different floor tiles, or adding decord on to walls to add additional variety and add a feeling of greater immersion
into the worlds your building. Have fun!
