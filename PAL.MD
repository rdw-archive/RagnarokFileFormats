# PAL Format Specification

## Overview

PAL files are simple color palettes used to share sets of related colors without having to redefine them repeatedly, and thus primarily serve to reduce memory (and disk) usage. For that same reason, the files don't include any header but rather are just a concatenated list of RGB values.

## Prerequisites

In order to understand the STR file format, some familiarity with the following concepts is required:

* [Color palettes](https://en.wikipedia.org/wiki/Palette_(computing)), a simple lookup table for RGB(A) colors

## Layout

All palettes (that I've seen) include 256 colors and are therefore 1024 bytes in size. Since there's no header or versions, that's about all there is to this format.

**TODO: Are colors stored as ARGB, RGBA, or ABGR?**