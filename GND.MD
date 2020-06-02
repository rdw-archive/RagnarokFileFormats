# Introduction

GND files contain the entirety of a map's static geometry (i.e., excluding objects and the water plane). The basic layout of the terrain is described as height vectors in 3D space, and with this information one can reconstruct the cubes/tiles and textured surfaces to generate and render the ground mesh.

**TODO: Visualisation of this concept**

Additionally, they include the pixels of a pre-baked lightmap which can similarly be assembled to render the shadows and static light sources of the ground. Those represented by the 3D models are cast by dynamic lights and listed in the map's RSW file instead.

For more information about the differences between the various light types in the game, see [the article about Light Sources and Ambient Light]().

**TODO: Link**

## Tiles and Cubes and Walls, oh my!

In the context of Ragnarok Online, there are many different terms floating around for the various concepts related to map geometry. To aid in understanding, let's go over them quickly.

### Tiles

[**Tiles**](https://en.wikipedia.org/wiki/Tile) are the basic building block for the terrain, so in a way RO could be described as a [tile-based game](https://en.wikipedia.org/wiki/Tile-based_game). It's not technically accurate, but that's definitely how the end result is perceived. Characters are positioned (at the center) on a tile, map coordinates refer to the tile and everything that requires thinking about positions usually works with the tile as the smallest unit.

In practical terms, the actual size of each tile as rendered on the screen depends on the scale of the world, camera zoom level and screen resolution. However, it's the baseline for measuring distance in the world coordinate system, where one unit of distance can be expressed in "number of tiles".

As far as textures are concerned, a texture appears to fit on a single tile if its dimensions are 32 pixels, as can easily be seen by checking the size of the ["grid selector"](https://i.imgur.com/GBuVjXe.png) texture rendered on top of the currently selected tile.

### Surfaces

**Surfaces** are the visible areas containing tiles. The entirety of the map's geometry is made out of surfaces, which can be textured or "invisible" and each surface contains exactly two tiles.

This is mostly of relevance for texturing, since texture slices usually are 64 pixels wide, so larger surfaces that appear seamless in the game are frequently pieced together by individual surfaces to give the appearance of a continuous textured plane (and they're indeed rendered similarly). Unfortunately this can cause problems if texture coordinates aren't applied correctly (and sometimes even if they are), which is why there are visible "gaps" in the terrain that can be seen at the right camera angle and distance.

### "Cubes"

Generally speaking, the terrain is defined by a heightmap indicating the positions of all four corners for any given tile. Each chunk of the heightmap (2x2 tiles) is sometimes called **cube**, which implies the presence of up to 6 surfaces: Two for the top and bottom of the cube, two to either side, and two to the front and back, where of course any number of them could be missing.

Now, the reason the cube metaphor doesn't work perfectly is that each chunk is only defined by *three* surfaces at most: Top, north, and east. If interpreted as surfaces it doesn't really change anything; Since you can express a cube's left (or south)-facing side as the adjacent cube's right (or north)-facing side you get the same effect with a much smaller data structure.

And that's pretty much how it works, as Gravity clearly went above and beyond to reduce file sizes in most of their assets. A side effect is that the northernmost "cubes" of each map can't have a "northern" wall, nor can the westernmost "cubes" have a wall to the west. This is why every single map has a specially-crafted "border" beyond the actual visible "walls" (if there are any), which are black impassable tiles that are just sitting there and can't usually be seen in the game, as they blend in with the background.

**TODO: Add screenshots to better explain this**

Usually, at least the top surface or one of the sides is there, as there'd be no point in defining an empty cube.

In the GND files, you will merely find each corner's height and the overall scale factor, which is enough to render the geometry. You could do this by translating the positions to world coordinates and normalize the heights so they refer to a cube of width and height 1 (as in, "one unit in world coordinates"), or just make the cubes bigger to account for the scale factor and then "zoom out" farther to get the same perspective.

The geometry is then rendered by simply glueing all the tiles together, and walls are implicitly formed by two tiles that aren't on the same height: If there's a height difference there'll be a plane in between the two tiles; this essentially becomes a "wall" if assigning it a texture. However, this is only done if the neighbouring "cube" is higher or lower and has a textured surface of its own

## Diffuse Colors (AKA Vertex Colors)

In order to give highlights to the terrain without requiring vast amounts of slightly different textures, the client can render specific (solid) colors at the corners of each tile to give this corner (and all adjacent ones) a different hue.

Again they massively saved on file size by providing only one color, which is applied to the bottom left corner (vertex) of each tile and the vertices of all adjacent tiles that happen to be in the same position. With this, one can define colored spots on the corners of the tile grid with very little effort or overhead.

**TODO: Screenshot**

This concept is known as vertex colors and it works by submitting additional information for a given point in 3D space that is then used by the hardware/GPU software to color it differently. It was widely used in older games where dynamic lighting wasn't feasible due to technical limitations, and it can have a significant effect for barely any effort at all:

In [this basic rendition of pay_dun00](](https://i.imgur.com/HWrzkEE.png)) you can see just the textured terrain with vertex colors applied (lightmaps, models, etc. are not displayed). They're the "blotches" on the ground, which blend in with the rest of the scene after all other effects are applied and give the terrain far more variety.

