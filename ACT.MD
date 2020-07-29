# ACT Format Specification

## Overview

ACT files store the animation data for all actions of an ingame entity, compiled as a single (binary) file. They're also used for static objects that are rendered using a spritesheet, i.e. anything that's rendered from an SPR file and not a regular image/BMP file.

They encode the following information:

* Available actions, such as "idle", "attack", "pickup" (looting an item), or "flinch" (getting hit)
* Animation frames, consisting of individual layers
* Sound effects (referenced), to be played during certain phases of the animation

Most notably, there are ACT files for items and even skill icons, which don't usually have an animation but still are assigned both an SPR and an ACT file regardless. Since there also exist regular bitmaps for them in the client files, I expect that these SPR/ACT files are only really needed for rendering equipment on characters, as well as lootable items in the game world (in short, anything part of the 3D environment).

Static skill or item icons displayed in the UI overlays could then simply use the actual images, which would make processing somewhat easier, though wasting disk space doesn't seem like something the developers would do if the other file formats are anything to go by.

While information from the SPR file is used to generate the actual bitmaps (images), the animation data allows the client to render objects in various phases of each animation, using the bitmaps contained in the SPR spritesheet.

Therefore the easiest way to think about ACT files is to simply consider them animation metadata, defining what images to display when and for how long for any given action. For static objects the only action then would be "idle", which means they aren't animated at all.

## Status

Mostly complete

## Open Questions

* Incomplete information on anchor points (they have an unknown field containing seemingly random data, not sure if it's actually used though)
* Are there more anchor points than one (players) or zero (creatures)? I've only seen these two, where the one existing anchor point refers to the body and is used to align the head, weapon, shield, headgear sprites.
* Are the inconsistent frame timings ignored by the client? If they aren't overridden, visible glitches occur, so presumably they just messed up the data for some frames (e.g., compare the [timings for rogue_female.act](ACT_InconsistentFrameTimings.png ) )

I've been able to render sprite animations correctly, so I figured it'll do for now.

## Prerequisites

In order to understand the ACT file format, some familiarity with the following concepts is required:

* The [SPR](SPR.MD) file format, which is tightly coupled to the interpretation of ACT files
* Basic [animation techniques](https://en.wikipedia.org/wiki/Animation#Techniques) and how they can be used to animate sprites

## Layout

The file structure differs slightly with each file version, as newer versions added more features. Please consult the tables below for a detailed specification.

It appears that (similar to SPR files) the header encodes the version in reverse, i.e. ``41 43 05 02`` reads as ``AC 5.2`` but it's actually version 2.5 and not 5.2.

The following versions are said to exist:

* 0x101 (to be confirmed)
* 0x200 (to be confirmed)
* 0x201 (to be confirmed)
* 0x203 (to be confirmed)
* 0x204 (to be confirmed)
* 0x205 (confirmed)

**TODO: Add file layouts for the different versions.**

I have yet to find any version other than just 2.5 to research them.

## Actions

Each action represents an activity the ingame entity (monster, player, homunculus, NPC) can perform. To be precise, it represents an action that can be *animated*; it's perfectly possible to have entities perform actions that *can't* be animated, as is evident by immobile monsters, such as plants, mushrooms or Pupa, attacking players and even moving without having a proper animation.

This is usually the result of a glitch or explicit serverside scripting forcing this behaviour and quite clearly unintended. However, in general the different entity types have animations for all of the relevant ingame actions.

Some ACT files contained in the client have glitched animations, e.g. they are too slow, have missing frames, or contain incomplete and unused actions. (**TODO: Examples/screenshots?**)

For example, there exists a "crying" animation for Rocker and an animation where Fabre is morphing into a Pupa. These can never seen ingame without modifying the client and possibly the server, too.

### Player Characters

The following animated actions have been observed:

* Idle
* Walking
* Sitting
* Picking up an item
* Standby (used after combat)
* Casting
* Attacking, in three different flavours (unarmed/armed with different weapon types)
* Two "frozen" animations (in combat, standing)
* A flinch (getting hit/taking damage) animation
* One featuring the player dead on the floor

### Monsters

The following animated actions have been observed:

* Idle
* Walking
* Attacking, in three different flavours (unarmed/armed with different weapon types)
* A flinch (getting hit/taking damage) animation
* One featuring the monster dying

Additionally, some monsters feature other animations, most of which I've never really seen ingame.

### Homunculi

They use the same animations as monsters, but (usually?) come with three attack animations that are probably used for the different skills? (**TODO: Confirm**)

### NPCs

They are usually static, though there's at least one (unused?) Kafra NPC that also has a walking animation.

### Item and Spell Icons

They are completely static, and their "standing" animation is an image that never changes. Some appear to have copy/pasted their standing animation into other actions, but clearly they weren't intended to actually be animated.

## Frames

Each animatable action consists of frames. A frame defines the actual animation, and consists of a number of *layers* as well as the animation delay and optionally a sound effect.

The animation for a unit's current action will proceed to play these frames with the given animationDelay from first to last, playing any sound effect for the given frame. only one can be played and subsequent sound effects would override those already playing.

The delay is given in units of 25ms each, so an animationDelay of 1 means 25ms in between each frame. A sound effect index of -1 means that there is no sound. Otherwise, the index refers to a sound file listed in the ACT, which is loaded separately and not part of the file itself.

Depending on the type of action, frames can be repeated. Standing, standby or walking animations are always looping, while attack, cast and dying (for monsters) are not. Frozen/dead animations could be implemented either way, as it doesn't make any difference.

## Layers

A layer is essentially the metadata defining each individual sprite instance, and multiple can be stacked on top of each other to generate complex effects. Most animations feature only a small number, or even just one, but casting effects often can assemble many smaller effects also contained in the SPR spritesheet to form advanced visual effects.

The information contained tells the client what part of the spritesheet to display, how to scale or position it, tint or mirror it, and may even rotate the image. It's probably easiest to see for yourself how they work using Tokei's GRF Editor or ActEditor.

### Invalid (unused) layers

It can be observed that some sprites seem to contain "empty" layers, which might be leftovers from an earlier version of the game, or were left in there for some other reason. These layers reference sprites with index ``-1``. These are presumably skipped by the client and not rendered, nor animated, in any way.

Example: [ACT_EmptyLayersExample.png](Images/ACT_EmptyLayersExample.png)

In the head sprite for male characters, the first layer appears to be an invisible copy of the second. I've not found any information as to the purpose of this, but it doesn't appear to be relevant to rendering the sprites correctly.

## Sound Effects

As mentioned above, playing multiple sound effects in a row will cut off those already playing. This means that idle frames are needed to play sounds without animations if a longer sound is desired and there aren't enough frames otherwise.

Apparently, headgear sprites cannot contain sound effects and the client will just ignore them if added manually?

## Scaling and you - 3D vs. 2D rendering layers

The following quotes summarize the issue with scaling sprites in the client:

> The original client does not seem to support the operations (scaling and rotation) inside the act format when the sprite is being projected onto a window. So if he scales the sprite images up and reduces the size inside the act format, all sprite images in the login interface (which might not be important) and others (equip window, skill window, etc.) will have huge sizes.

> This is why changing the ACT file to use upscaled sprites in Gravity's client inevitably causes the equipment/talent tree UI to display a vastly oversized sprite. Unfortunately, this cannot be easily "fixed" given the technological limitations.

Essentially, there are two different layers with different technical limitations:

* The 2D UI layer, which does not implement sprite scaling
* The 3D layer, which supports scaling (based on each layer's scaleU/V properties)

The 2D layer contains all interface frames and therefore also item icons displayed in the player's inventory, as well as the character preview in the equipment and talent tree windows.

The 3D layer contains all sprites seen in the game world, as well as items dropped to the floor (which also live as entities in the game world). I believe it also includes all effects, even those that are basically sprites and purely 2D.

## ACT vs IMF (char select window)

The character selection screen appears to be even weirder; it's not really on the UI layer and therefore seems to use special files in the IMF format to display sprites.

This is probably an artifact of some weird design remnant of the early alpha/Arcturus times, so I wouldn't worry about replicating or even fixing this.

## SpriteTypes

Apparently, there are only two sprite types defined in each ACT, and they seemingly refer to the type of the spritesheet image (SPR file): Indexed palette/BMP (type 0) and RGBA/TGA (type 1)? (**TODO: Needs more research**)

## Attachments

This is where the fun begins! As you may be aware, character sprites consist of different parts, like head, headgear, body, weapon, shield, cart, and they're all different sprites and therefore stored in different ACT/SPR files.

How the client stitches these together is very much akin to Frankenstein's monstrosity, and there are layering issues to be considered, too. The weapon sprites, for example, must be drawn behind or before the body based on the character's direction.

According to various sources, there exists offsets in some ACT files that allow the client to correct these attachment's positions, but I haven't researched this further.

The exact calculation is clearly convoluted, and I only found one source giving more details on this. I have translated the picture they gave here from the Chinese original, with a little help from a friendly Redditor:

![](Images/coordinatesCorrectionEN.png)

Note that the Y axis is as usually inverted, so the offsets must be interpreted accordingly. Since I still haven't implemented the sprite attachments myself, so I can't give more details on how accurate the calculations are at this time. (**TODO: Update this part once sprite rendering is fully understood**)
