# RSM Format Specification

## Overview

RSM files contain data used to render 3D objects in the game world. They are mostly reserved for architecture, trees, etc. and aren't normally used for units, with the exception of WOE Guardians and the Emperium.

## Prerequisites

In order to understand the RSM file format, some familiarity with the following concepts is required:

* [Polygon meshes](https://en.wikipedia.org/wiki/Polygon_mesh), used to represent objects in 3D space
* [Bone animation](https://en.wikipedia.org/wiki/Skeletal_animation), which is (presumably?) the technique used to animate 3D meshes

## Nodes

* Position and nodes? What about the coordinate system? Left/bottom corner = origin?

## Hierarchy

* A hierarchy of nodes, parent/children?

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
