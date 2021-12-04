# GR2 Format Specification

GR2 files contain 3D models and skeletons stored in the proprietary [Granny3D](http://www.radgametools.com/granny.html) format.

This article only covers the files used in Gravity's RO client and does not aim to be a comprehensive resource for all possible variants and versions of Granny3D files.

## Prerequisites

Before learning about Granny3D files (as used in RO), you'll probably want to read this first:

* The (much simpler) [RSM](RSM.MD) format description, which uses keyframe 3D animations directly
* [Skeletal animation](https://en.wikipedia.org/wiki/Skeletal_animation), a technique for animating 3D models via "bones" and "weights"
* [B-Splines](https://en.wikipedia.org/wiki/B-spline#Computer-aided_design_and_computer_graphics), a way of compactly storing animation data as mathematical curves
* [Endianness](https://en.wikipedia.org/wiki/Endianness), a term that describes the architecture-dependant byte order of data
* [Marshalling](https://en.wikipedia.org/wiki/Marshalling_(computer_science)) can be employed to store data in an architecture-independent fashion
* [The Wikipedia article on Data Compression](https://en.wikipedia.org/wiki/Data_compression), for an intro to compression algoritms

## Overview

GR2 files are used very sparsely in RO, and appear to be a (failed) experiment that took place late in the game's development (2003): They are only used to represent WOE Guardians, the Emperium as seen during WOE, castle flags, and treasure boxes. They also contain an unfinished version of what looks to be a 3D player model (skeleton and textures only).

There are two "types" of Granny3D files used in RO: The model and idle pose of each 3D actor and can be found in files located in the ``data/model/3dmob`` directory, while the others each store a skeleton for another pose and are found in ``data/model/3dmob_bone``.

Unlike the custom formats used in RO, Granny3D files cannot as easily be deserialized. This is because they use a proprietary compression algorithm called [Oodle](http://www.radgametools.com/oodle.htm), which comes in many different versions and is (unfortunately) not widely supported by existing open-source tools.

Each Granny file encodes a hierarchy of objects and some metadata, with all data being stored in containers called sections. The compression is applied to (some of) the data sections, while the data stored inside, like textures and animations, can also be compressed (potentially using a different algorithm). This makes processing them somewhat difficult.

There are ways to get at the data, of course, but they generally rely on specific format conversion (such as a GR2-to-DAE converter that I've seen) or may not be fully compatible with all possible GR2 versions (various open source projects for other games come to mind).

## Features

Granny files can store arbitrary data, but in the case of RO they alway seem to follow a relatively primitive "standard" format. In particular, they don't appear to extend the basic Granny file format with custom data types. This means only some of the vast Granny3D feature set is used, the relevant portions of which should be described here.

### Mixed Marshalling

Going by what's been implemented in various open-source decompressor tools, the format allows storing data in an architecture-dependent form. This may necessitate a preprocessing step where it's updated for the current computer architecture (to restore the endianness).

It seems that this feature is indeed used by some of the files (e.g., ``aguardian90_8.gr2`` features 4 mixed marshalling blocks). I haven't looked into the specifics as OSS tools exist that will handle the de-marshalling process and I therefore consider it a "solved problem".

### Compression

Variant: Oodle 0 (Granny compression mode 1; 0 is uncompressed - so it must be the oldest?)

Some sections are uncompressed (only the final one for each file?)
**TODO: Verify this by decoding all 21 files and checking the last section**

**TLDR:** LZMA-style compression scheme, probabilistic, stateful. TBD: Is this a modified variant? Sounds like it (described by authors as ``lzhuff``).

**TBD**: Outdated version of the algorithm? There's a string in ``granny2.dll`` that says this:

**TBD:** How is it different from the other (newer) Oodle implementations? Does it matter? (The current ones are available as open source/on GitHub, but Oodle0/1 aren't?)

![A message that seems to indicate Oodle0 is no longer supported](Images/GR2_OodleString.png)

**TODO: Check RAD changelog for details about the various versions**

### Sections

Granny files consist of multiple buffer areas that the decompression needs to be applied to individually before a proper and more useful Granny File structure can be obtained.

Once all sections have been decompressed and re-arranged, the resulting struct still requires further processing in order to retrieve the stored skeletons, meshes, and textures.

** TODO: Rearrangement not needed, section references suffice?**

Textures (and animation curves?) are also compressed, though I'm unsure whether they use the same Oodle0 compression as the sections themselves or some other, related algorithm (Bink video?).

**TBD:** Why is the data split this way? Perhaps to differentiate between persistent and temporary resources (vertex data -> goes to GPU, then can be freed. What parts do you want to keep? Animations and skeletons?)

**TODO: Research texture compression and the compression of animation curves**

**TODO: Is anything else compressed or was that it?**

### Other Features (TODO: Determine which are used)

Not sure yet, but there are many features that need to be checked:

* Material data
* Texture compression (Bink video?)
* Mip levels (are they even used? RO doesn't know any modern LOD AFAIK)
* LOD culling (unused?)
* Spline Animations
* Root Bone Motion?
* Combining skeletons from two different files (since they split idle/combat poses)
* Screenshot of Zombie Dragon? (not really important)

## Layout

I have verified that all 21 GR2 files in the RO client use version 6 of the Granny file format.

| Field | Offset | Size | Type | Description and notes              |
| :-----: | :------: | :----: | :-----: | :---------------------------: |
| Header | 0 | 352 bytes | struct | The Granny File header structure (see below) |
| Sections | 352 | variable | binary | Compressed Granny File data, split into multiple parts |
<!-- | TBD | TBD | 4 Bytes | TBD | TBD |
| TBD | TBD | 4 Bytes | TBD | TBD | -->

The decompressed sections need to be relocated; it looks as though they are simply appended at the end of the header section:  The ``relocationOffset`` is always ``352``, which is the length of the header structure.

### Granny Header Structure

While the header clearly allows for flexible structures, in Ragnarok's GR2 files the size is fixed:

| Field | Offset | Size | Type | Description and notes |
| :-----: | :------: | :----: | :-----: | :---------------------------: |
| Signature  |   0    | 16 Bytes | binary | Always ``B8 67 B0 CA F8 6D B1 0F 84 72 8C 7E 5E 19 00 1E``               |
| Header Size | 16 | 4 Bytes | uint32 | Always ``352`` (decimal) |
| Compression Flag | 20 | 4 Bytes | uint32 | Always ``0`` (no compression) |
| Unknown | 24 | 4 Bytes | ? | Unused (probably) since it's always zero |
| Unknown | 28 | 4 Bytes | ? | Unused (probably) since it's always zero |
| Version | TBD | 4 Bytes | uint32 | The Granny File version (always ``6``) |
| File Size | TBD | 4 Bytes | uint32 | The total file size in bytes |
| Checksum | TBD | 4 Bytes | uint32 | A CRC checksum over the file contents |
| Section Offset | TBD | 4 Bytes | uint32 | Where the sections begin (always ``56``) |
| Section Count | TBD | 4 Bytes | uint32 | Number of sections (always ``6``) |
| Root Node Type | TBD | 8 Bytes | struct | Points to a definition of the top-level node object's type |
| Root Node Object | TBD | 8 Bytes | struct | Points to the top-level node object in the data tree |
| User Tag | TBD | 4 Bytes | uint32 | Always ``0F 00 00 80`` |
| User Data | TBD | 16 Bytes | ? | Unused (probably) since it's always zero |

### Section Reference Structure

Objects referenc the decompressed section buffer area using this simple format:

| Field | Offset | Size | Type | Description and notes |
| :-----: | :------: | :----: | :-----: | :---------------------------: |
| Index | 0 | 4 Bytes | uint32 | The index of the section containing this object |
| Offset | 4 | 4 Bytes | uint32 | Where the object is located inside the section |

### Decompressed File Structure

**TODO: Needs more information/validation of preliminary results**

## See also

There are various open source projects providing info on Granny files:

* [https://github.com/arves100/Granny2-research/wiki/File-Format-Documentation](https://github.com/arves100/Granny2-research/wiki/File-Format-Documentation)
* [https://github.com/Karbust/gr2decode](https://github.com/Karbust/gr2decode)
* https://github.com/herenow/gr2-web
* https://github.com/Norbyte/lslib/


Some information about Oodle0 (from the people at RAD) can be found here:

* [http://www.radgametools.com/oodle.htm](http://www.radgametools.com/oodle.htm)
* [http://www.radgametools.com/oodlecompressors.htm](http://www.radgametools.com/oodlecompressors.htm
)
