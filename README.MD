**This repository will no longer see updates.** My work continues here: [Ragnarok Research Lab](https://github.com/RagnarokResearchLab/)

To see the latest results of my research into Ragnarok Online's file formats, visit [the new documentation website](https://ragnarokresearchlab.github.io/file-formats/).

Note: I'll archive the repo after the migration of all relevant contents has been completed.

---

# Ragnarok Online File Formats Manifest

Gravity Co has used some fairly arcane file formats in creating their first MMORPG client, presumably supported by custom (in-house) or now-obsolete software.

This repository contains my findings from researching how they work and how they can be used or modified, alongside sources and links that at times have been quite difficult to track down after all these years.

Hopefully having it all collected in a git repository will allow the information to remain available for many years to come. Feel free to do with this as you please!

Obvious disclaimer: I don't claim ownership of any findings and results of the hard work that others have put into analysing these file formats. Sometimes it's impossible to say who originally discovered the various details due to the passage of time and obscure references I had to consult, but I'll list my sources as best as possible.

## Status

I've analysed and created tools to work with most of the common file formats by now. However, there are many unknowns and smaller details that may still need clearing up in order to understand the formats perfectly.

Often these are likely to be just oddities stemming from the fact that Ragnarok Online uses the old Arcturus engine, so they might not be required to process the files and render their contents correctly.

Newer (post-Renewal) changes may not be reflected in this specification, though I expect to look into them more once the basics are covered.

## Contributing

Please open an issue or contact me directly via Discord (RDW#9823) if you can contribute in any way! Hopefully, with some help from the community most of the unknowns can be eliminated eventually.

## File Types Overview

All of the file types come in different versions, because *of course they do*. Usually, newer versions add more features while many of the older versions are virtually unused, being remnants from the early days of RO development and Gravity's previous project, [Arcturus: The Curse and Loss of Divinity](https://tcrf.net/Arcturus:_The_Curse_and_Loss_of_Divinity).

They used a custom, in-house engine that was later extended to create Ragnarok Online. It's also most probably why RO's file formats and architecture seem pretty odd, by modern standards.

However, since even the earliest RO clients don't support all of the legacy formats, they will be of limited interest. As existing tools aren't really compatible, researching them further isn't something I'm currently planning on.

### World Data (Map files)

The most complex data stored and processed by the client is arguably the world and environment data. There are several files that are combined to create a representation of the world (maps) and its parameters, which however are not perfectly understood at this time.

There exist several third-party programs that appear to recreate a faithful copy of the original client's rendering output, so the information available should be enough to interpret the original world without visible deficits.

| Extension | Interpretation              | Contents                                                       |
| --------- | --------------------------- | -------------------------------------------------------------- |
| GND       | Ground | Map geometry, texture references, lightmap data     |
| GAT       | Altitude and Terrain | Terrain altitude and type (ground, cliff, water)               |
| RSW       | World Resources | 3D objects, water, lights, audio sources, effect emitters |

### Sprites (2D Actors)

Most characters, monsters, and NPCs are displayed as 2D sprites with their various animations defined as "animation clips", which consist of multiple images. One clip exists for each of the different view angles, separating a full circle into just eight basic directions.

Which direction should be used is determined by the client based on the camera position, i.e. the angle between the camera and the monster's position, and the assumed direction it is currently facing (I think). The latter is probably derived from its movement pattern, since I didn't find any serverside code that would manage this.

The information on how to display a given unit's ingame actions is embedded in special collection files that contain additional information, such as sound effects to be played as part of the animation.

Animations are composed of simple image files that will be played as individual frames, consisting of one or several layers that can be arranged to display multiple sprites together, in a predefined order, and with various visual changes applied to them (such as rotation, scaling, transparency, and color shading).

| Extension | Interpretation       | Contents                                                              |
| --------- | -------------------- | --------------------------------------------------------------------- |
| SPR       | Spritesheet          | Sprite images as bitmaps or TGA, metadata and, optionally, an RGB palette     |
| ACT       | Animation Collection | Keyframes, animation details, and, optionally, a list of sound effects to be played |
| IMF       | Interface Management File | Information used to render sprites on the UI (2D) layer (seems obsolete?)|

### Models (3D Objects and Actors)

Very few 3D models have been used for actual monsters, and the formats are somewhat unfamiliar to me. Most 3D models are those placed on the map while rendering and represent objects in the environment, like trees or architecture.

Since the format for these appears to be custom or vastly outdated, I haven't found any standard tools to edit them, but people have written software to at least display the models and render them as part of a given map.

| Extension | Interpretation       | Contents                                                              |
| --------- | -------------------- | --------------------------------------------------------------------- |
| RSM       | Model | 3D models used for objects (and sometimes units) in the game world    |
| GR2       | GRANNY2 | 3D models stored in an ancient format once used by the [Granny3D](https://radgametools.com/granny3d) framework |
| RSX       | ?| Arcturus 3D model format? (doesn't appear to be used in RO) |

### Effects

Effects come in several varieties, all of which are processed separately. Some effects simply use the existing ACT/SPR formats, some use a separate compiled (binary) format, some are hardcoded into the client executable and in later versions particle effects can also be generated from settings stored in specific Lua files.

The hardcoded effects appear to be limited to effects involving 3D components, such as particles and geometric primitive shapes, though they will often use sprites and other textures as part of their animation. Purely sprite-based animations are provided as separate files, which are mostly understood by now.

Particles are configured using the "effect tool" and only seem to be used in recent years, with individual configuration files defining effect emitters for a given map.

| Extension | Interpretation       | Contents                                                              |
| --------- | -------------------- | --------------------------------------------------------------------- |
| STR       | ? | Compiled (binary) effects format |
| EZV       | ? | Raw (text-based) effect format? |

### Script and configuration files

Client-side data and Mercenary/Homunculus AI is written in the Lua programming language and embedded in the client itself, or sometimes as text-files with roughly-CSV-like structure (where fields are separated by hash tags).

Lua files are human-readable text files and thus easily understood, even if the creators clearly didn't have readability in mind. LUB files are precompiled Lua files and can be somewhat reversed. This allows one to read much of them, though some information is still lost in the process.

Thankfully, LUB files are only used in Post-Renewal clients (or so it seems), which means that all the classic files are perfectly readable. Converters exist to restore the original table structure from LUB files, though I cannot vouch for their correctness.

| Extension | Interpretation       | Contents                                                              |
| --------- | -------------------- | --------------------------------------------------------------------- |
| LUA       | Lua source code | Lua script file |
| LUB       | Lua bytecode | Precompiled Lua script |

### Audio files (Music and Sound Effects)

Finally, something easy! These are just regular WAV or MP3 files, respectively, and can be played with any standard program.

### Interface files

They are regular images (bitmaps). Nothing to be said about them, other than that the Korean/Unicode filenames don't normally translate properly on non-Korean Windows systems and therefore can end up as gibberish.

Only 256 colours are supported and the first palette index is used as transparent background colour. Usually this is the oldschool "pink" with RGB code (``#FF00FF``), though similar colours can also be occur:

> It's safe to clear anything with red and blue > 0xf0 and green < 0x10

Guild emblems are stored as compressed bitmaps, in the [EBM](EBM.MD) format.

## Community Resources

See [here](CommunityProjects.md) for a list of editing tools and client reimplementation projects that may help with understanding the file formats described in this documentation.

## Endianness

All numbers found in RO's binary formats are stored as [little-endian](https://en.wikipedia.org/wiki/Endianness), which means that the individual bytes have to be "reversed" before interpreting the binary data as a number.