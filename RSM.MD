# RSM Format Specification

RSM files contain data used to render 3D objects in the game world. They are mostly reserved for things like buildings, trees, and even hunter traps.

## Prerequisites

In order to understand the RSM file format, some familiarity with the following concepts is required:

* Basic [string representations](https://www.cs.yale.edu/homes/aspnes/pinewiki/C(2f)Strings.html) (both counted and delimited strings are used)
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



### Version 1.1

This is [claimed to exist](https://github.com/flaviojs/eathena-devel-FlavioJS/blob/master/client/file_formats/rsm.txt), though I've never seen it.

### Version 1.2

This is claimed to:

* Add the color parameter to texture vertices
* Add the smoothing group property to triangle faces

Since there are no earlier versions to compare this one to, this can't be verified.

### Version 1.3

This is said to alter the format of the bounding boxes, adding an unknown boolean flag. It's sensible to assume that this would affect the collision detection somehow, most likely either simplifying it (for performance reasons) or making it more accurate.

### Version 1.4

This is said to add transparency (flavioJS), but Tokei writes that it isn't used ingame.

### Version 1.5

Adds scale keyframes to each node (appears to be unused)

### Version 2.2

* Animation duration is now given in frames per second
* Each model can have multipe root nodes
* Adds translation key frames to each node
* Strings are stored with their length instead of fixed-length with a null terminator
* The initial placement is stored as a transformation matrix and baked into the animation keyframes, instead of being computed from the individual transformation components

### Version 2.3

* Textures are referrenced for each mesh, not globally
* Adds texture animations to each node

## See also

* [Tokei's explanation of the RSM2 changes](https://rathena.org/board/topic/127587-rsm2-file-format/)