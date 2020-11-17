# STR Format Specification

## Overview

STR files are a compiled (binary) form of animated, image-based effects used to display certain kinds of 2D effects in the game world. According to the author of RoBrowser, they are simply EZV compiled to binary (**TODO: Verify**).

In conjunction with hardcoded 3D effects that use geometrical primitives and Direct3D textures directly, as well as regular sprites (stored in SPR/ACT files), different types of effects can be realized.

They encode the following information:

* References to texture files (bitmaps)
* Data used to animate each layer
* Animation properties applicable to all textures

## Prerequisites

In order to understand the STR file format, some familiarity with the following concepts is required:

* It's recommended to read the [SPR](SPR.MD) and [ACT](ACT.MD) specifications first, as sprite animations share many key concepts with STR effects
* [Keyframe animation](https://en.wikipedia.org/wiki/Key_frame) is essentially what's being used by both sprites and effects, in a slightly different fashion

## Effects vs. Sprites

While storing effects in a separate file format seems handy, they differ from SPR and ACT files (that can also be abused to create effects) in that they don't contain the actual image data. Instead, they reference files locally, i.e. inside the ``data/texture/effect`` folder. This is where both the STR files and the bitmaps they use are stored.

From a technical POV, it appears that sprites are simply meshes rendered in billboard mode (always facing the camera) and then are modified to account for camera perspective, while STR effects are rendered either as projected textures (for AOE/ground effects) or similarly as "actors" in the game world but without the complex logic required to handle look/face directions. More research is needed to confirm this, however.

## Layout

The file structure differs slightly with each file version, as newer versions added more features. Please consult the tables below for a detailed specification.

**TODO: Note the version byte order**

**TODO: Layout for the different versions**

## Tools

There are only a few tools available to parse STR files, as far as I know:

* [RO STR Viewer](https://herc.ws/board/topic/17268-rostrviewer-reupload): Looks a bit dated, but contains all the information as far as I can tell
* https://github.com/skardach/ro-str-viewer: I actually didn't care enough to make it run, but an open-source reference implementation is always nice to have
* RoBrowser (see [Tools.md](Tools.md)) has an Effect viewer that appears to be fully functional