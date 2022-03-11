# RSM Format Specification

RSM files contain data used to render 3D objects in the game world. They are mostly reserved for things like buildings, trees, and even hunter traps.

## Prerequisites

In order to understand the RSM file format, some familiarity with the following concepts is required:

* [Polygon meshes](https://en.wikipedia.org/wiki/Polygon_mesh), used to represent objects in 3D space
* [Smoothing groups](https://en.wikipedia.org/wiki/Smoothing_group), a way to implicitly define surface normals
* 3D keyframe animation, where animation keyframes are stored as matrices

## Overview

The RSM format seems to be the standard format for 3D models used by Gravity, and it supports most of the features one would expect from a 3D format: Geometry, texturing information, and even a keyframe-based animation system.

Meshes are defined as a hierarchy of nodes, with transformations being applied to the tree in a cascading fashion in order to compute the final position of each contained object (in local space). These meshes are then positioned according to the instancing properties defined in the RSW file for a given map (in world space).

It should be noted that, while models can be animated, they aren't used to represent moving agents in the game world, like players or enemies, i.e., they can't "move". Their use lies exclusively in representing decorative scene objects ("props" or "doodads").

Nowadays, there are two versions of RSM files, with major differences and incompatibilities between them. The newer models all use the ``.rsm2`` file extension, while the original versions always use ``.rsm``.

It also appears that the older versions have been (repeatedly) altered, without updating the version number. This leads to conflicting information and a general sense of confusion when it comes to the exact details of the various properties; a situation that, unfortunately, I've not managed to completely untangle so far.

## Nodes

Each model contains (at least) one root node, and an arbitrary number of child nodes. The latter can be nested multiple times, and usually define smaller parts of a larger model. For example, the root node of a "windmill" model could be the base of the building, while the individual wheels would be defined in separate child nodes.

Often, these will be smaller moving parts that are animated independently and need to be transformed correctly to form a cohesive whole. In the original version, only one such construction could be stored in the RSM file, defined by its root node, but it seems RSM2 files also support storing several independent root nodes at once.

---

The rest of this article is unfinished :( Proceed at your own peril.

## Animations

Seems like they can be chained = concatenated matrix multiplications (just basic Linear Algebra/transformations, I guess)?

* Animation keyframes

Not the same kinds of keyframes as used by 2D animations, but rather "snapshots" that are then interpolated?

> During disassembling original client from end of 2016 i found that all the info about rsm format that i had found is wrong. It seems that koreans removed position animation from 1.5. Okay, i written their behavior, but i still has extra 8 bytes at the end of file and i cannot figure out are they unused in original client or not. There's no extra bytes in 1.4 for example.

Verify this? See Aesir source and compare the various implementations/do some testing?

## Transparency

> I've used BrowEdit to compare my results and found an issue that was the same as in my project. Some models had wrong depth writing. Using the correct order, it is possible to render objects regardless of their transparency. You can try to open BrowEdit and compare dicastes01 with this image.

What image? Did I save it somewhere? Hmm. Not sure what's with the depth writing, research later?

> In Arcturus you could interact with certain 3D models (e.g. doors, chests), so C3dActor was a part of the game object hierarchy. In Ragnarok, all 3D models are just decoration and not part of the game play and aren't considered game objects. (The exceptions to the rule are Granny actors and traps, the latter of which are actually RSM models drawn by a Skill object.)

RSM models drawn by a Skill object? That sounds so crazy, it might just be true.

> Yeah you have to calculate bounding boxes to display rsm correctly on the map.

Will have to see, so far I've only rendered the terrain (GND/ground mesh). The bounding box wasn't really needed for that.

## Bounding Boxes

* Information about the bounding boxes, used for collision detection


> Looking at your issue, you might want to take a look at the code here: https://github.com/Doddler/RagnarokRebuild/blob/master/RebuildClient/Assets/Scripts/MapEditor/Editor/RagnarokModelLoader.cs

> Around line 340 is where it's centered. The short of it is that you have to recalculate the bounds for the model and it's children and recenter it on load. I do it myself by calculating the bounds of the full model and then, from it's local position, subtracting the min of the bounds y axis and the center of the bounds x and z axis (assuming y is up).
> 
*Source: Doddler*

## Duplicate faces

> Some RSM files apparently have duplicate faces

Do they, now? What a surprise (...not). How are they treated by the client?

# Shading

> The RSM now support is natural shadding type (without, flat, or smooth) instead of using smooth shadding on all models, result: better render, faster loading.

## Layout

**TODO: Add tables for the different versions**

**TODO: Version byte order**

### Version 1.1

### Version 1.2

* Vertex color? Texture color? Smoothing group?

### Version 1.3

* Bounding box flag?

### Version 1.4

* Alpha

### Version 1.5

See Aesir?

> From version 1.5 and above each node may have individual key frames for its animation.

### Version 2.0

See GRF Editor?

### Bounding Boxes

**TODO:  Are the bounding boxes actually used? If so, how?**

RJS got some of them wrong, but it doesn't appear to cause visible glitches. See the mushrooms in the bottom left corner of pay_dun00, for example
