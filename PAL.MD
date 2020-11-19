# PAL Format Specification

## Overview

PAL files are simple color palettes used to share sets of related colors without having to redefine them repeatedly, and thus primarily serve to reduce memory (and disk) usage. For that same reason, the files don't include any header but rather are just a concatenated list of RGB values.

## Prerequisites

In order to understand the PAL file format, some familiarity with the following concepts is required:

* [Color palettes](https://en.wikipedia.org/wiki/Palette_(computing)), a simple lookup table for RGB(A) colors


## Transparency

Transparency is determined for each color according to its RGBA values, after loading the palette:

	transparentColor = { alpha = 0, red = 10, green = 10, blue = 10 }

	if color.red >= 0xFE and color.green < 0x04 and color.blue >= 0xFE then
		color = transparentColor
	else
		color.alpha = 0xFF
	endif

This hack may be explained by the fact that not all of the bitmaps in the client use the exact same background color (there's different shades of pink).

## Packed Palettes

Only the leftmost 5 bit of each color value are actually stored, and the rest is truncated. The alpha component is stored as a single bit flag (transparent or not), as defined by the conversion algorithm above.

The implications are clear (pun intended): Palette colors can't use opacity since the alpha value is discarded, so they're either fully visible or entirely transparent based on the color used.

Additionally, similar colors will become identical if they differ only in the last 3 bit. I'm not sure if the difference is perceptible or if those color values are even used at all, but it's something to keep in mind as the client won't be able to render them correctly.

## Layout

There's very little to say:

* All palettes contain 256 colors
* Colors are given in 8-bit ARGB format
* The entire palette is therefore 1024 bytes in size

Since there's no header or versions, that's about all there is to this format.