# GND Format Specification

## Overview

GND files contain the entirety of a map's static geometry (i.e., the ground/terrain). It doesn't include objects and the water plane, which are stored as part of the RSW file instead.

## Prerequisites

In order to understand the GND file format, some familiarity with the following concepts is required:

* [Polygon meshes](https://en.wikipedia.org/wiki/Polygon_mesh), used to represent objects in 3D space
* The [Flyweight Design Pattern](https://gameprogrammingpatterns.com/flyweight.html) can help to reduce space usage by avoiding redundancy
* [Lightmaps](https://en.wikipedia.org/wiki/Lightmap), a standard way to use precomputed lighting and improve rendering performance

## "Flyweight" compression

The basic layout of the terrain is described in terms of height vectors in 3D space, and identical surfaces are re-used frequently. The list of unique surfaces comes with a reference to the used texture path for each surface, and each block of terrain in turn references one unique surface for each of its sides. With this information one can reconstruct the cubes/tiles and textured surfaces to generate and render the actual ground mesh.

While this representation is far less intuitive than the standard ways to define a mesh, it does save some space. The savings must've been worth the trade-off at the time, though it makes it far more difficult to understand (and work with) the format.

The reconstruction process is rather involved, and requires building the mesh by combining all visible (textured) surfaces in just the right way while resolving the various lookups. Since coordinates are implicit the absolute positions must be calculated by adding the origin to each cube, account for relative positioning based on the cube's index, determine which "walls" to show by comparing the surfaces of neighbouring cubes, and finally scaling everything correctly.

## Tiles, Cubes, and Surfaces

RO uses tiles as the basic unit of measurement for game logic, but its world actually consists of surfaces larger than that, which are part of what you could call "cubes" (boxes). These boxes are combined like a jigsaw puzzle to form the actual terrain.

Each surface of a cube is 2x2 tiles large (**TODO: Are there any exceptions to this rule?**) and may be textured or not. There's no direct reference to the tiles as defined in the world coordinate system, but rather surfaces/cubes are combined in a predefined order (bottom/left to top/right).

Cubes are defined by only three surfaces (TOP, NORTH, EAST) which are then mirrored, if visible, at the opposite direction (BOTTOM, SOUTH, WEST). The implication is that terrain cubes can't be assigned different textures on the two sides sharing a surface reference.

Since this is far from simple, more details on the individual parts follow below.

### Tiles

As previously mentioned, [tiles](https://en.wikipedia.org/wiki/Tile) are the basic building block for the terrain, so in a way RO could be described as a [tile-based game](https://en.wikipedia.org/wiki/Tile-based_game). It's not technically accurate, but that's definitely how the end result is perceived. Characters are positioned (at the center) on a tile, map coordinates refer to the tile and everything that requires thinking about positions usually works with the tile as the smallest unit... except when it comes to rendering.

In practical terms, the actual size of each tile, as rendered on the screen,depends on the scale of the world, camera zoom level and screen resolution. However, it's the baseline for measuring distance in the world coordinate system, where one unit of distance can be expressed in "number of tiles".

As far as textures are concerned, a texture appears to fit on a single tile if its dimensions are 32 pixels, as can easily be seen by checking the size of the ["grid selector"](https://i.imgur.com/GBuVjXe.png) texture rendered on top of the currently selected tile.

### Surfaces

Surfaces are the visible areas containing tiles. The entirety of the map's geometry is made out of surfaces, which can be textured or "invisible" and as far as I can tell, each surface contains exactly four tiles (two in each dimension).

This is mostly of relevance for texturing, since texture slices usually are 64 pixels wide, so larger surfaces that appear seamless in the game are frequently pieced together by individual surfaces to give the appearance of a continuous textured plane (and they're indeed rendered similarly). Unfortunately this can cause problems if texture coordinates aren't applied correctly (and sometimes even if they are), which is why there are visible "gaps" in the terrain that can be seen at the right camera angle and distance.

### Cubes

Generally speaking, the terrain is defined by a heightmap indicating the positions of all four corners for any given tile. Each chunk of the heightmap (2x2 tiles) is sometimes called cube, which implies the presence of up to 6 surfaces: Two for the top and bottom of the cube, two to either side, and two to the front and back, where of course any number of them could be missing.

Now, the reason the cube metaphor doesn't work perfectly is that each chunk is only defined by *three* surfaces at most: Top, north, and east. If interpreted as surfaces it doesn't really change anything; Since you can express a cube's left (or south)-facing side as the adjacent cube's right (or north)-facing side you get the same effect with a much smaller data structure.

[This visualization](Images/GND_SurfacesVisualization.png) might help understand the implications. In the picture, surfaces are colored as follows:

* Green if they're the TOP tile for a given cube
* Blue if they're NORTH
* Red if they're EAST

What this means is that, counter to intuition, the blue parts to the south of each chunks aren't really the south surface of the respective ground cube, but rather the north one of the neighbouring cube to the south. The "walls" are implicitly defined by the difference in both cubes' height vector and the client renders them according to which of the top tiles is higher, or not at all if the surface's aren't textured.

 A side effect is that the northernmost "cubes" of each map can't have a "northern" wall, nor can the westernmost "cubes" have a wall to the west. This is why every single map has a specially-crafted "border" beyond the actual visible "walls" (if there are any), which are black impassable tiles that are just sitting there and can't usually be seen in the game, as they blend in with the background.

In the GND files, you will merely find each corner's height and the overall scale factor, which is enough to render the geometry. You could do this by translating the positions to world coordinates and normalize the heights so they refer to a cube of width and height 1 (as in, "one unit in world coordinates"), or just make the cubes bigger to account for the scale factor and then "zoom out" farther to get the same perspective.

The geometry is then rendered by simply glueing all the tiles together, and walls are implicitly formed by two tiles that aren't on the same height: If there's a height difference there'll be a plane in between the two tiles; this essentially becomes a "wall" if assigning it a texture. However, this is only done if the neighbouring "cube" is higher or lower and has a textured surface of its own.

## Diffuse Colors (AKA Vertex Colors)

In order to give highlights to the terrain without requiring vast amounts of slightly different textures, the client can render specific (solid) colors at the corners of each tile to give this corner (and all adjacent ones) a different hue.

Again they massively saved on file size by providing only one color, which is applied to the bottom left corner (vertex) of each tile and the vertices of all adjacent tiles that happen to be in the same position. With this, one can define colored spots on the corners of the tile grid with very little effort or overhead.

**TODO: Screenshot**

This concept is known as vertex colors and it works by submitting additional information for a given point in 3D space that is then used by the hardware/GPU software to color it differently. It was widely used in older games where dynamic lighting wasn't feasible due to technical limitations, and it can have a significant effect for barely any effort at all:

In [this basic rendition of pay_dun00](https://i.imgur.com/HWrzkEE.png) you can see just the textured terrain with vertex colors applied (lightmaps, models, etc. are not displayed). They're the "blotches" on the ground, which blend in with the rest of the scene after all other effects are applied and give the terrain far more variety.

