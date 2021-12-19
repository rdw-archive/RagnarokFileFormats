# EBM Format Specification

EBM files contain compressed bitmaps and are used exclusively to represent guild emblems.

## Prerequisites

There really isn't much to read up on here:

* [Bitmap (BMP)](https://en.wikipedia.org/wiki/Bitmap_file_format) is one of the standard image formats used in the RO client
* [DEFLATE](https://en.wikipedia.org/wiki/Deflate) is a standard compression algorithm and serves to reduce the file size on disk

## Overview

EBM files are what the game creates when players upload regular image files as their guild emblem. They are always stored in the ``_tmpEmblem`` folder, which I'd call "emblem cache".

The emblems are decompressed and can then be displayed in only a few places:

* Next to the character and guild name when hovering over a player character's sprite
* In the Guild management UI that also allows uploading new emblems
* On the guild flag model, when a guild has conquered the corresponding castle

## Compression

The client simply applies the ``INFLATE`` (for decompressing) and ``DEFLATE``(for compressing) standard algorithms, provided by [zlib](https://en.wikipedia.org/wiki/Zlib), to the raw bitmaps to convert from/to EBM.

This makes obtaining the original image file trivial as long as zlib can be used directly:

```javascript
const fs = require('fs');
const zlib = require("zlib");

const SOME_EBM_PATH = "myEmblem.ebm";

const buffer = fs.readFileSync(SOME_EBM_PATH);
const bitmap = zlib.inflateSync(buffer);

fs.writeFileSync(SOME_EBM_PATH + ".bmp", bitmap);
```

*Example: Decompressing EBM files using [Node.js' builtin zlib module](https://nodejs.org/api/zlib.html)*

## See also

* The [GR2 specification](GR2.MD) includes information about displaying emblems on guild flags
