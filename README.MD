# Ragnarok Online File Formats Manifest

Gravity Co has used some fairly arcane file formats in creating their first MMORPG client, presumably supported by custom (in-house) or now-obsolete software.

This repository contains my findings from researching how they work and how they can be used or modified, alongside sources and links that at times have been quite difficult to track down after all these years.

Hopefully having it all collected in a git repository will allow the information to remain available for many years to come. Feel free to do with this as you please!

Obvious disclaimer: I don't claim ownership of any findings and results of the hard work that others have put into analysing these file formats. Sometimes it's impossible to say who originally discovered the various details due to the passage of time and obscure references I had to consult, but I'll list my sources as best as possible.

## File Types Overview

### World Data (Map files)

The most complex data stored and processed by the client is arguably the world and environment data. There are several files that are combined to create a representation of the world (maps) and its parameters, which however are not perfectly understood at this time.

There exist several third-party programs that appear to recreate a faithful copy of the original client's rendering output, so the information available should be enough to interpret the original world without visible deficits.

|  Extension | Interpretation  | Contents |
|---|---|---|
| GND  | Ground Tile Mesh  | Map geometry (polygons), texture references, lightmap data |
| GAT | Ground Altitude Data | Terrain altitude (height and flag for walkable tiles) |
| RSW  | World Resources | Map structure (models/objects to render on top of the terrain)

Note:
Some (all?) of these file types seem to come in different versions, because *of course they do*. I haven't yet figured out the exact differences, but I'll update the respective entries when I do.

### Sprites (2D Actors)

All characters, monsters, and NPCs are displayed as 2D sprites with various animations consisting of multiple images for different directions (usually mirrored, so only half of the possible directions have unique representations).

These may be embedded in collection files that contain additional information, such as sound effects and animations, but technically they're all just simple image files that will be played as individual frames in a predefined order.

Some collection files contain additional sprites, such as shadows (?) or overlay effects, but I haven't been able to find out more about how they are used exactly.

### Models (3D Objects and Actors)

Very few 3D models have been used for actual monsters, and the formats are somewhat unfamiliar to me. Most 3D models are those placed on the map while rendering and represent objects in the environment, like trees or architecture.

Since the format for these appears to be custom or vastly outdated, I haven't found any standard tools to edit them, but people have written software to at least display the models and render them as part of a given map.

### Effects (Animations)

It should be said that there are MANY effects that were hardcoded into the client itself (sigh). Unfortunately, I was unable to find out much more about this, but it appears to be limited to simpler effects such as particles flying. All the complex animations are provided as separate files, which are mostly understood by now.

Simpler effects also seem to be created by manually animating 2D sprites, so that often only a BMP or TGA image file exists and no compiled (binary) animation file that can be used directly. This ties in with how many effects are hardcoded in the client... and it could potentially make recreating them a challenging task.

### Script files (Lua and LUB)

Client-side data and Mercenary/Homunculus AI is written in the Lua programming language and embedded in the client itself, or sometimes as text-files with roughly-CSV-like structure (where fields are separated by hash tags).

Lua files are human-readable text files and thus easily understood, even if the creators clearly didn't have readability in mind. LUB files are precompiled Lua files and can be somewhat reversed. This allows one to read much of them, though some information is still lost in the process.

Thankfully, LUB files are only used in Post-Renewal clients (or so it seems), which means that all the classic files are perfectly readable. Converters exist to restore the original table structure from LUB files, though I cannot vouch for their correctness.

### Audio files (Music and Sound Effects)

Finally, something easy! These are just regular WAV or MP3 files, respectively, and can be played with any standard program.

### Interface files

They are regular images (bitmaps). Nothing to be said about them, other than that the Korean/Unicode filenames don't normally translate properly on non-Korean Windows systems and therefore can end up as gibberish.

Only 256 colours are supported and the first palette index is use as transparent background colour. Usually this is the oldschool "pink" with RGB code (``#FF00FF``).

## Sources

* [Ximosoft's Ragnarok Online Laboratory](http://rolaboratory.ximosoft.com): Link is now defunct, but can be accessed via the Wayback Machine/Internet Archive
* http://mist.in/gratia/ro/spr/ActFileFormatFix.html
* http://mist.in/gratia/ro/spr/SprFileFormat.html
* https://rathena.org/board
* HerculesWS
* RoBrowser (dysfunctional)
* Various open-source client implementation attempts (all of which are now abandoned)
* OpenKore wiki
* http://openkore.com/index.php/Field_file_format