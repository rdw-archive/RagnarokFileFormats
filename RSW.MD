# RSW Format Specification

## Overview

RSW files essentially represent a collection of visible and invisible game objects used to fill the 3D world with life. They include information about:

* The map's water plane
* Light sources and global illumination
* Environmental audio sources
* 3D models (architecture)
* Effect emitters

## Prerequisites

In order to understand the RSW file format, some familiarity with the following concepts is required:

* Coordinates and objects in [(Euklidian) 3D space](https://en.wikipedia.org/wiki/Euclidean_space)
* Light and audio in 3D space (dynamic light sources and spatial audio sources)
* How to use simple meshes ("water planes") to simulate fluid surfaces (**TODO: Link/explanation is in order?**)
* Effects (in Ragnarok), see [EZV](EZV.MD), [STR](STR.MD), and [ACT](ACT.MD)/[SPR](SPR) (**TODO: Effects vs. Effect emitters, Lua vs. RSW emitters**)
* [Bounding Boxes](https://en.wikipedia.org/wiki/Minimum_bounding_box), which are often used for collision detection and [frustrum culling](https://en.wikipedia.org/wiki/Hidden-surface_determination)
* [Scene Graphs](https://en.wikipedia.org/wiki/Scene_graph) and [Quad Trees](https://en.wikipedia.org/wiki/Quadtree), data structures used to represent the hierarchy of objects in a 3D scene and store the bounding boxes of objects (**TODO Relationship needs clarification?**)

## The Water Plane

**TODO: RSW 2.2 - Water animation seems to have become more complex? Unknown field, no one seems to have studied it yet**

I'm not 100% on this part as I haven't yet implemented the wave effect myself. It looks like there are a few factors to consider, however.

### Water Types

Each map has exactly one assigned "water type". These simply define what texture to use and provide the prefix to the water texture's file name, which is appended with the cycling ID to determine which of the individual frames to display.

Not all of them are obviously water; there's also some "mud" and "lava" textures that use the same mechanism (described below) to be animated.

### Texture Cycling

A moving water surface is simulated by increasing a cycling counter each frame and updating the water mesh's texture when a cutoff point (**TODO: Which one?**) is reached, with textures cycling from 0 to 31.

Each water type has these 32 textures and they can be found in the ``textures/effects/water`` (in Korean?) folder as separate images.

A full cycle then takes ``3 * 32 = 96 frames / (60 frames / second) = 1.6 seconds`` at the default cycling speed of ``3``, which is also set in the map's RSW file, assuming a fixed framerate of 60 FPS (which is clearly what they were counting on).

### Ground Tiles and the Water Plane

The water plane itself appears to not cover the entire map area, as one would expect from a naive approach. Instead, each visible ground tile (GND cube's TOP surface) is assigned a "water vertex", which is the position of the water plane for this corner of the tile.

The X and Z coordinates are always pinned to the tile's X and Z coordinates, while the Y coordinate is used to define the water level at this corner of the tile.

If the water is still, all four corners will be at the same height, which is exactly the water level as defined in the map's RSW file.

### Wave Cycles

If there is a wave effect, the various wave parameters will be used to simulate physical waves in the following manner:

* The "wind" is thought to always blow in direction (1,1), meaning from bottom left to top right (from southwest to northeast)
* This means that the wave will hit the ground tile in this same direction, which is "faked" by setting the top left and bottom right corners to the same height and the bottom left and top right to a different one, each corresponding to the wave height at the respective time in the wave cycle
* Calculate the height of the wave [crest and trough](https://en.wikipedia.org/wiki/Crest_and_trough) for each part by the sine wave formula given above
* As the frames iterate through the wave cycle, the heights are modified to display the water mesh's vertices in a "wavy" pattern

**TODO Exact formula**

**TODO Add screenshots?**

This simulation updates every frame (so ideally, 60 times per second?). It is not triggered via a timer, so variances in frame rate should affect the speed of wave animations greatly.

Each update will increase the "time" of the wave cycle by a small, fixed amount, until the full cycle has ran its course and is reset. Subsequently, the "water vertices" are updated, altering their height at each visible tile only (to save on unnecessary calculations).

What tiles are visible appears to be determined by the ground mesh's bounding box/scene graph, as defined in the GND file?

**TODO: Verify**

## Global Illumination and Dynamic Lighting

Only the ambient light source is rendered by the client, while the directional  light sources are ignored and instead "faked" via the GND file's lightmap texture? (pre-baked lightning)

I'd assume that the scenes were lighted dynamically and the results then rendered to the lightmap texture. It's not clear why they chose to leave them in, though? **TODO: Research dynamic light sources**

## Ambient and Directional light sources

**TODO: Research type of the "ambient" light source. Is it ambient or actually directional?**

**TODO: Research the default values (used in older versions, see ASB dump)**

**TODO: Formula for obtaining the light direction (as vector) from latitude and longitude**

If lightmaps are disabled, the color values of the ambient light are multiplied by 1.5 (maxing out at 1.0, obviously). This is done presumably to make the map appear less dark and easier to see?

Do they use max (1.0) intensity? Clearly the RSW alpha is ignored, but what value DO they use? It's not in the ASB code

**TODO: ASB Code could shed some light on this, modify/port so it can be shared?**

> As far as I have tested, the lights in the RSW structure only affect objects, not the ground.

There were some shenanigans with lightning. Can we determine the rules of which objects are affected, and how, for the individual light sources?

## Object Types

Objectxx

Cylinderxx
Boxx
GeoSphereXX
MeshXX
SphereXX

That should be all, no? What's the point of having a generic "Object" type?

### 3D Models

**TODO: What is the render order? There was something written about it on one of the athena forums, but I can't find it.**

### Spatial Audio Sources

**TODO: Max audio channels?**

**TODO: Assassin's Guild entrance bug with "infinite" stacking audio sources may serve to illustrate the limitations?**

**TODO: Compare to ACT sounds (unit as a single audio source)?**

### Effect Emitters

Example: Ghost "sprites" in Niffleheim

**TODO: Compare RSW emitters to Lua-based ones?**

## Frustum Culling and the Scene Graph

Object hierarchy is stored in quadtree, but it's only present in version 2.1 and above?

**TODO: What are they doing for earlier versions?**

> BrowEdit: always use 586 as it has corrected quad tree calcs where 620 does not.

What's the difference? Compare diffs?

> However, the solution is pretty simple. The original client uses a quadtree to improve culling performance. Each model intersects or is within at least one quadtree node. If a model does not belong to a node, than it won't get rendered at all. Basically, those nodes are bounding boxes. The client gets all those boxes that are within the viewing frustum. Afterwards, the client checks which models belong to which quadtree node and renders them. The object in the screen does not belong to any node and therefore shall not be rendered.

> If you don't use a quadtree (or octree, whatever), then there is another simple solution: All quadtree nodes are (or have to be) within the xz-coordinates of the ground bounding box. So if a model has a position that is not within this range, then just don't render it.

Example: Amatsu Tree is culled (**TODO: Screenshots**)

> Another cool effect that was left out was having static models between the camera and the player disappear so that they don't hide the player. I'm not sure if the models were planned to be completely or just partially hidden. This feature was more or less complete, but the approach used is maybe to simple so that it doesn't always work as intended. I also discovered a bug in the calculation of the bounding box used especially for this purpose. You can actually try out this feature by using NEMO patcher, as I found a relatively simple approach to enabling it. Just enable "Restore model culling" when patching your client.

Investigate this?

More quad tree stuff:

>  The data associated with a leaf cell varies by application, but the leaf cell represents a "unit of interesting spatial information".

Here it's the 3D objects/bounding boxes?

> A tree-pyramid (T-pyramid) is a "complete" tree; every node of the T-pyramid has four child nodes except leaf nodes; all leaves are on the same level, the level that corresponds to individual pixels in the image. The data in a tree-pyramid can be stored compactly in an array as an implicit data structure similar to the way a complete binary tree can be stored compactly in an array.

Just a serialized struct containing all the bounding boxes, then?

> A region quadtree with a depth of n may be used to represent an image consisting of 2n × 2n pixels, where each pixel value is 0 or 1. The root node represents the entire image region. If the pixels in any region are not entirely 0s or 1s, it is subdivided. In this application, each leaf node represents a block of pixels that are all 0s or all 1s. Note the potential savings in terms of space when these trees are used for storing images; images often have many regions of considerable size that have the same colour value throughout. Rather than store a big 2-D array of every pixel in the image, a quadtree can capture the same information potentially many divisive levels higher than the pixel-resolution sized cells that we would otherwise require. The tree resolution and overall size is bounded by the pixel and image sizes.

We don't actually need this using modern frameworks, which will handle culling etc. But perhaps it can be visualized (as an image) to more easily explain the feature?

## Unused features

> For instance the world resource format (RSW) contains strings pointing to scripts files, a feature not used in Ragnarok

Dynamic lighting also goes here?

## Camera restrictions

> RSW table for indoor maps and implemented rotation range

### Layout

**TODO: Table for the different versions**

#### Version 1.6

Used in iRO alpha (circa 2003). More research is needed.

#### Version 2.1

Widely used in RO (Classic). Later (Renewal) superceded by slightly modified versions?
