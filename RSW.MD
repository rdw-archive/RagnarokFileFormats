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

### Global Illumination and Dynamic Lighting

Only the ambient light source is rendered by the client, while the directional  light sources are ignored and instead "faked" via the GND file's lightmap texture? (pre-baked lightning)

I'd assume that the scenes were lighted dynamically and the results then rendered to the lightmap texture. It's not clear why they chose to leave them in, though? **TODO: Research dynamic light sources**

### Ambient and Directional light sources

**TODO: Research type of the "ambient" light source. Is it ambient or actually directional?**

**TODO: Research the default values (used in older versions, see ASB dump)**

**TODO: Formula for obtaining the light direction (as vector) from latitude and longitude**

If lightmaps are disabled, the color values of the ambient light are multiplied by 1.5 (maxing out at 1.0, obviously). This is done presumably to make the map appear less dark and easier to see?

Do they use max (1.0) intensity? Clearly the RSW alpha is ignored, but what value DO they use? It's not in the ASB code

**TODO: ASB Code could shed some light on this, modify/port so it can be shared?**

### 3D Models

**TODO: What is the render order? There was something written about it on one of the athena forums, but I can't find it.**

### Spatial Audio Sources

**TODO: Max audio channels?**

**TODO: Assassin's Guild entrance bug with "infinite" stacking audio sources may serve to illustrate the limitations?**

**TODO: Compare to ACT sounds (unit as a single audio source)?**

### Effect Emitters

Example: Ghost "sprites" in Niffleheim

**TODO: Compare RSW emitters to Lua-based ones?**

### Frustum Culling and the Scene Graph

Object hierarchy is stored in quadtree, but it's only present in version 2.1 and above?

**TODO: What are they doing for earlier versions?**

Example: Amatsu Tree is culled (**TODO: Screenshots**)

### Layout

**TODO: Table for the different versions**

**TODO: Version byte order**