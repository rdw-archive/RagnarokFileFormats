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
| GAT | Ground Altitude and Terrain | Terrain altitude and type (ground, cliff, water) |
| RSW  | Resources and Water | Map structure (models/objects to render on top of the terrain)

Note:
Some (all?) of these file types seem to come in different versions, because *of course they do*. I haven't yet figured out the exact differences, but I'll update the respective entries when I do.

### Sprites (2D Actors)

All characters, monsters, and NPCs are displayed as 2D sprites with various animations consisting of multiple images for different directions (usually mirrored, so only half of the possible directions have unique representations).

These may be embedded in collection files that contain additional information, such as sound effects and animations, but technically they're all just simple image files that will be played as individual frames in a predefined order.

|  Extension | Interpretation  | Contents |
|---|---|---|
| SPR  | Spritesheet | Sprite images as bitmaps, metadata and the palette (colours used) |
| ACT | Animation Collection | Keyframes, animation details, and possibly sound effects to be played |


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

## Tools

Not all tools are equally useful, but you might be interested in seeing how they work with the various file formats if your desire is to learn more about them.

### CuriousKitten's GNDEdit

Highly recommended to view all the intricate map details; no tool I've found gives a more comprehensive overview. Unfortunately it doesn't explain what any of the information means, so it's not as useful as it could be and can be a bit overwhelming at first.

[Link @ dotalux.org (mirror)](https://dotalux.com/ro/GNDedit/)

### Tokei's GRF Editor

The most modern file viewer I could find, which even supports map rendering! Unfortunately, it doesn't seem to show the data for the complex structures that it parses, such as the GND's cube geometry itself.

[Link @ rathena.org](https://rathena.org/board/files/file/2766-grf-editor/)

### Borf's BrowEdit

It's a map editor, which allows you to edit maps. I don't think it's very stable and the UI is truly horrendous, but nevertheless there might be some situations where playing around with it can help you to understand the map structures better.

Since there's so many different versions out there and most of them seem to randomly crash, I can't give a specific link that is guaranteed to work. You can try compiling the source obtained [on GitHub](https://github.com/Borf/browedit) and hope for the best.

If someone can point me to a stable version of this I'll happily add the link. However, from what I've seen, such a version does not exist.

### Vincent Thibault's RO Browser

Not a tool by itself, this is an app designed to mimic the original Ragnarok client's behaviour, which you can run in your browser. In order to achieve this incredible feat, it had to deal with virtually ALL of the proprietary file types, which is probably why it was taken down by Gravity in the first place.

While I've no idea what happened there, the source code is once again available on GitHub and since it's written in JavaScript doesn't require any compilation. You can simply open the files and read what's inside of them, or change things around to see how it affects the game world.

So in a way, it is the ultimative tool to aid in understanding the file formats, or it would be if the source was more readily understandable and the things it does much better documented than is unfortunately the case.

[Link @ GitHub](https://github.com/vthibault/roBrowser)

## Sources

* [Ximosoft's Ragnarok Online Laboratory](http://rolaboratory.ximosoft.com): Link is now defunct, but can be accessed via the Wayback Machine/Internet Archive
* [The Hercules source code](https://github.com/HerculesWS/Hercules)
* [Vincent Thibault's RoBrowser](https://github.com/vthibault/roBrowser)
* [Gratia Huang's website](http://mist.in/gratia/ro/)
* Various open-source client implementation attempts
* [VCS's OpenKore wiki on field types](http://openkore.com/index.php/Field_file_format)