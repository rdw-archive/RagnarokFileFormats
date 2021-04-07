# Cloud Effects

There are different cloud/fog effects for each map, all which are hardcoded. Most of them are simple variations on the same pattern, changing only the movement speed/number of textures etc.

Like any other hardcoded effect, they use a primitive particle system, where textures are mapped to render primitives and the particles are then animated depending on a variety of parameters.

## Overview

Each cloud effect is created by managing several (many!) individual clouds, which in turn are comprised of (always?) four primitive (3D) effects, to which textures are applied in a randomized fashion. Their properties are then animated to create the illusion of movement/shifting clouds/more or less fog.

This usually creates only clouds, but in one map (Einbroch), it is used to create the appearance of "fog" instead. The fake "fog" is different from the actual DirectX fog effect, which is used to colour-tint the scene instead and isn't normally perceived as fog, unless one was to zoom out very far.

## Variations

The following maps use a texture-supported cloud effect:

| variationID |     mapID      | effectID | numClouds |
| :---------: | :------------: | :------: | :-------: |
|      0      |   (multiple)   |   229    |    40     |
|      1      |    ``yuno``    |   230    |    60     |
|      2      |  ``valkyrie``  |   233    |    40     |
|      3      |  ``einbroch``  |   515    |    80     |
|      4      |  ``airplane``  |   516    |    80     |
|      5      | ``thana_boss`` |   592    |    80     |
|      6      | None (unused)  |   ---    |    ---    |
|      7      |  (New World?)  |   697    |    80     |
|      8      |  (New World?)  |   698    |    80     |

### Observatory Platforms (Variation 0)

I believe variation 0 is used for the flying platform on ``gef_fild07``, as it has a cloud effect when entered. Geffen was also added to the game much earlier than Yuno or Einbroch, and the IDs were clearly added in chronological order.

None of the earlier maps, like Prontera or Morocc, have any cloud effects whatsoever. There's also a similar platform on ``mjolnir_01``, which probably uses the same effect, and must've been added at around the same time.

The interesting part here is that clouds only show when the player moves on top of the platform, and they are hidden when descending.

### New World (Variations 7 and 8)

The last two variations could be anything released after (or alongside) Thanatos Tower:

* Abyss Lake (nope)
* Hugel (nope)
* Rachel (no clouds here)
* Veins (maybe? I don't remember this place much. Don't think so, though)
* Nameless Island (no clouds, AFAIK)

It could also be any of the Renewal location that came later:

* El Dicastes
* Bifrost
* Eclage
* All the other stuff (I don't know any of these maps)

Variations 7 and 8 were added approximately at the same time as a skill ``ALL_PARTYFLEE`` (``Blowing Wind``? Can't find much info on this), and the item effect for something called ``Huge Spray of Flowers``.

This would put it near Mora Village? I have no idea about Renewal content so it might be totally off... I'd then expect these two effects two be used in any of the New World areas.

Since the properties aren't altered and use the same version that Yuno does, all that's left is to find out *where* they are used (mapID) to enable the effect.

**TODO: I guess it would be possible to naively search all these maps by warping there and manually check where fog is used?**

### Other maps using the same presets

There are also a number of other maps which, according to the RoBrowser implementation, use the same variations:

* ``airplane_01`` (same as ``airplane``)
* ``gonryun`` and ``gon_dun02`` (unknown preset. Looks like ``valkyrie`` or ``yuno``?)
* ``himinn`` (same as ``valkyrie``)
* ``ra_temsky`` (also ``valkyrie``)
* ``rwc01`` (kRO doesn't have this map, so no idea)
* ``sch_gld`` (looks like a ``yuno`` clone)
* ``5@tower`` and ``6@tower`` (I've no clue what this is)
* ``archer_21`` (I don't have any info on this)

## Lifecycle

In RoBrowser, clouds are particles that spawn randomly, and quickly die. There's always 150 of them and they die after 8 seconds. I don't think this is accurate, but it seems "close enough".

In Gravity's client, particles have a "birth" and "lifetime" (essentially an `isAlive`` flag). For short-lived particles, like the hit or spell animations, they'll quickly fade. Cloud particles, however, don't cycle and are instead permanent - when their practically infinite lifetime "ends" it is simply reset.

Once a map is loaded, the cloud effect is triggered and a varying number of clouds (based on the preset configuration for the map) is created. Each cloud consists of a fixed number of instances that are overlaid.

These clouds move in a circle (rotation) and simultaneously towards the sky, while their opacity changes, causing a "wavy" movement pattern to manifest. It appears as though clouds are "dying" and being re-created. In reality they're always the same particles, but are relocated to stay near the player, and fade out periodically before fading back in again.

## Fog/Cloud Textures

Each unique cloud particle is randomly assigned a texture based on the preset configuration:

* For ``einbroch``: Uses texture ``effect/fogX.tga`` (1, 2, 3)
* Anything else: ``effect/cloudX.tga`` (1, 2, 4; 3 is unused?)

Einbroch is a special case since it uses the cloud effect to generate the appearance of industrial smog.