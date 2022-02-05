# RSW Format Specification

## Overview

RSW files essentially represent a collection of visible and invisible game objects used to fill the 3D world with life. They include information about:

* The map's water plane
* Light sources and global illumination
* Environmental audio sources
* 3D models (architecture)
* Particle effect emitters

## Prerequisites

In order to understand the RSW file format, some familiarity with the following concepts is recommended:

* [Bounding Boxes](https://en.wikipedia.org/wiki/Minimum_bounding_box), which are often used for collision detection and [frustrum culling](https://en.wikipedia.org/wiki/Hidden-surface_determination)
* [Quad Trees](https://en.wikipedia.org/wiki/Quadtree#Region_quadtree), a data structure that is here used for the purposes of culling objects from the scene
* [Particle systems](https://en.wikipedia.org/wiki/Particle_system), since particle emitters are frequently encountered in RSW files

If you want to understand the water plane's wave animations, try these links:

* You'll need a decent understanding of [waveforms](https://en.wikipedia.org/wiki/Waveform), specifically [sine curves](https://en.wikipedia.org/wiki/Sine_wave) and the [unit circle](https://en.wikipedia.org/wiki/Unit_circle#Trigonometric_functions_on_the_unit_circle)
* Some other terms may also pop up in this article: [Phase shift](https://en.wikipedia.org/wiki/Phase_(waves)#Phase_shift), [amplitude](https://en.wikipedia.org/wiki/Amplitude), and [wavelength](https://en.wikipedia.org/wiki/Wavelength#Sinusoidal_waves)
* It would certainly help to know about keyframe animations and sampling (can't find a good link)

## The Water Plane

In the most commonly-used versions of the format, information about the map's water can be found. This is used to build a special "water plane" mesh, the vertices of which are animated according to a waveform curve in order to simulate waves, which are always tied to the ground mesh's geometry.

It should be noted that there is no actual fluid simulation or wind effect, or anything resembling a typical physics engine being used in RO; the systems are far more primitive and probably hand-rolled.

As of the latest version, this information is no longer present. It is instead found in the maps' GND file.

There's also a secondary "texture cycling animation", which simply replaces the texture image for each water surface with the next one, similar to how sprites are animated by cycling through a texture atlas.

### Water Types

Each map has exactly one assigned "water type". These simply define what texture to use and provide the prefix to the water texture's file name, which is appended with the animation frame ID to determine which of the individual frames to display, while cycling through all 32 frames.

Not all of them are obviously water; there's also some "mud" and "lava" textures that use the same mechanism (described below) to animate.

### Texture Cycling

A moving water surface is simulated by increasing a cycling counter each frame and updating the water mesh's texture after it has been displayed for a sufficient duration, with texture indices cycling from 0 to 31. This number appears to be fixed, as all water types have exactly 32 frames.

A full cycle then takes ``3 * 32 = 96 frames / (60 frames / second) = 1.6 seconds`` at the default cycling speed of ``3``, which is also set in the map's RSW file, assuming a fixed framerate of 60 FPS (which is clearly what they were counting on). I'm guessing this all would break down if the FPS  drop.

### Ground Tiles and the Water Plane

The water plane itself appears to not cover the entire map area, as one would expect from a naive approach. Instead, each visible ground tile (GND cube's TOP surface) is assigned a "water vertex", which is the position of the water plane for this corner of the tile. These vertices effectively form the water plane described above; which isn't necessarily a continuous surface.

The X and Z coordinates are always pinned to the tile's X and Z coordinates, while the Y coordinate is used to define the water level at this corner of the tile. If the water is still, all four corners will be at the same height, which is exactly the water level as defined in the map's RSW file.

### Wave Cycles

If there is a wave effect, the various wave parameters will be used to simulate physical waves in the following manner:

* The "wind" is thought to always blow in direction (1,1), meaning from bottom left to top right (from southwest to northeast), in map coordinates, i.e. (x, z)
* This means that the wave will hit the ground tile in this same direction, which is "faked" by setting the top left and bottom right corners to the same height and the bottom left and top right to a different one, each corresponding to the wave height at the respective time in the wave cycle
* Calculate the height of the wave [crest and trough](https://en.wikipedia.org/wiki/Crest_and_trough) and map them to the tile's corners
* What I call the "left" trough is [phase-shifted](https://upload.wikimedia.org/wikipedia/commons/9/92/Phase_shifter_using_IQ_modulator.gif) to the left of the crest, the "right" trough to the right
* As the frames iterate through the wave cycle, the vertex heights are modified to display the water mesh's vertices in a "wavy" pattern

### Animation Sampling Basics

This simulation updates every frame (so ideally, 60 times per second?). It is not triggered via a timer, so variances in frame rate should affect the speed of wave animations greatly.

Each update will increase the "time" of the wave cycle by a small, fixed amount, until the full cycle has ran its course and is reset. Subsequently, the "water vertices" are updated, altering their height at each visible tile only.

What tiles are visible appears to be determined by the same mechanism used to cull objects from the scene, by utilizing the quad tree of regions and their bounding boxes, as defined in the GND file.

### Waveform Sampling Algorithm

The computation itself is a bit involved, but here is essentially how it works:

1. Compute the wave sampling offset ("animation frame index")
2. Compute the relative offsets of the wave troughs from the crest
3. Map the troughs and crest to the corners of each ground tile, accounting for its map position
4. Sample a sine curve at this offset to calculate the surface height at the crest and troughs
5. Scale the offsets and account for the plane's water level to determine the final height

To help visualize this, one can imagine a repeating wave pattern moving over the water surface, in the implied wind direction of (1, 1), i.e. north-east, with each trough and crest being exactly one tile apart:

![Assets/Images/RSW_WaveTroughsAndCrest.png](Assets/Images/RSW_WaveTroughsAndCrest.png)

*Pictured: A testament to my incredible graphs editing skills (waves advance in direction north-east)*

The tiles are implied and form a single GND surface (4x4 GAT tiles):

![Assets/Images/RSW_WaveTroughsAndCrestWithGrid.png](Assets/Images/RSW_WaveTroughsAndCrestWithGrid.png)

As the imagined wind blows (the wave animation cycle proceeds), the troughs and crests of the waves (red and orange lines, respectively) move towards the north-east and the water vertices are deformed at the tile corners according to the sampled height value. You'll have to imagine the waves in my illustration since this is all terribly low-budget, but I'm sure you'll see the pattern ingame this way.

If that wasn't exactly clear, and I'm almost certain it wasn't, I recommend checking out some ingame maps and observing the water plane's movement as it interfaces with the landmass.

In excruciating detail, the algorithm would look something like this (not sure if this actually helps...):

* Compute the wave cycle offset, as coordinates on the unit circle (in degrees)
* Starting at zero, compute the next sampling offset for the wave crest by adding the phase offset
* This happens once per frame (60 times per second?); imagine a clock ticking once per frame
* With each "tick" of the clock, the offset moves around the unit circle
* Whenever the animation loops around (reaches 180 degree), simply reset to -180 degree
* Offset this by the position (in map coordinates) of the GAT tile the water vertex is assigned to
* Additionally, phase-shift the trough offsets to move them left/right of the crest
* This can be imagined as two additional hands of the clock (left and right of the original one)
* Modulate a sine wave with the parameters from the RSW file and the above cycle offsets
* This is again computed separately for each of the four corners of the GAT tile
* During the above step, the crest is mapped to the bottom-right and top-left corners
* The left trough is mapped to the bottom-left corner, with an offset of -1
* The right trough is mapped to the top-right corner, with an offset of 1
* Lastly, the final phase shift is obtained by factoring in the surface curvature (from the RSW)
* The curvature angle therefore acts as an amplifier to increase the phase shift (if nonzero)
* If you now "unwrap" the unit circle into a line from -180 to 180, take the offsets as x-values
* Multiply in the phase shift, add the tile position offsets and sample a standard sine curve
* FInally, multiply by the amplitude parameter (scaling factor) to determine the wave's height
* The resulting height values are the final offsets of the wave's crest and both troughs at the corner
* Those sampled height offsets are added to the water plane's Y-offset (the baseline water level)

All of this only applies to maps that actually have a wave animations. When the waveform amplitude (local rate of change per sampled interval) is zero, there is no animation and the water plane's surface remains still, with the height simply being the water level (wave offset is zero).

### Waveform Visualization

Since this is all very difficult to understand without visualizing  the waveforms and trying out different values, I've prepared a spreadsheet you can play around with: [Download it here](Assets/WaveCycleVisualizationSheet.xlsx) and give it a try!

PS: Requires MS Excel :( I didn't think to use Google Sheets or LibreOffice in time, so here we are.

*Disclaimer: I made this only for myself and didn't originally intend to publish it, but then figured I might as well, so don't expect too much. Also, yes, I know git wasn't made to store binary files, but this repository shouldn't be missing anything of relevance so I added it anyway.*

To help illustrate the effect of adjusting the various waveform parameters and their impact on the computed water surface over time, here's some graphs (made with the above spreadsheet):

![Assets/Images/RSW_WaveOffsets_alberta.png](Assets/Images/RSW_WaveOffsets_alberta.png)
*Pictured: Surface curvature over time for ``alberta.rsw`` (medium-sized waves are repeating every 2 seconds)*

![Assets/Images/RSW_WaveOffsets_comodo.png](Assets/Images/RSW_WaveOffsets_comodo.png)
*Pictured: Surface curvature over time for ``comodo.rsw`` (large waves are cycling every 3 seconds)*

![Assets/Images/RSW_WaveOffsets_aldebaran.png](Assets/Images/RSW_WaveOffsets_aldebaran.png)
*Pictured: Surface curvature over time for ``aldebaran.rsw`` (the water barely moves; each cycle takes 6 seconds)*

It should be noted that the above graphs depict *only* the curvature, and not the final offset (height of the water plane's vertices). It is used as an input in the final step of the computation, as described above, but the water surface closely mirrors the sine curve as it's only scaled by the amplitude parameter afterwards.

### Renewal Water Changes

As mentioned, in the latest (Renewal-only) RSW and GND versions there have been some significant changes. For details, check out the [GND specification](GND.MD), which includes my findings on the matter.

In terms of wave cycles, I don't believe the fundamentals have changed. I'm assuming the wave animation mechanism simply extends to each of the water planes assigned to a map, i.e., there would be several animations running in parallel (if they aren't all identical).

All of this still requires further research as it can't easily be verified without access to the kRO servers.

### Transparency

Since the terrain beneath the water plane is always shining through, I'm guessing there's some transparency being used for the water planes. It's hard to tell how it works exactly, but my "best guess" estimate based on manual testing is that it could be around a 50% opacity (alpha value of ``0.5``):

![Assets/Images/RSW_WaterPlaneAlpha.png](Assets/Images/RSW_WaterPlaneAlpha.png)

*Pictured: Overlaying water textures with zero and 50% transparency, respectively*

More testing needs to be done to verify this, particularly in combination with the wave animations.

### Environmental Light Influences

It looks like the environmental light may also influence the color of the water plane somehow. This is mostly noticeable when it's very dark, such as in the case of ``mag_dun01``: The "water" (lava) texture is quite bright, but the ingame rendition looks rather dim - in alignment with the rest of the scene.

If there was indeed no such influence, the texture should be as bright as the textured ground mesh surfaces, where in reality it's similar in color to the "walls" and shadows (determined by ambient light).

Therefore I'm (again, guessing here) that the ambient color of the scene light might be factored into the water plane's diffuse color, though more testing needs to be done to verify (or disprove) this.

It's quite possible this is just another artifact of them applying some sort of filter. As the colors of all third-party-renditions I've seen look noticeably off, it has been postulated that there's some magnifying (or dampening) of the lighting, e.g. a bloom filter or some other post-processing effects.

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

Used in iRO alpha (circa 2003), as well as in Gravity's previous game, Arcturus. More research is needed.

#### Version 2.1

Widely used in RO (Classic). Later (Renewal) superceded by slightly modified versions?
