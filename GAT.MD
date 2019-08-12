# Gound Altitude Data

GAT files contain two types of information primarily: The altitude data for each tile of a map, and the block type. The latter is used to determine whether or not a given tile is walkable or can be sniped through by actors in the game world, which is only somewhat related to its altitude  (too high = can't snipe nor walk over it, I guess?).

All numbers are stored as [little-endian](https://en.wikipedia.org/wiki/Endianness), which means that the individual bytes have to be reversed before interpreting the binary data as a number.

## Layout

This is the structure of each GAT file as far as I was able to determine:

| Field | Offset | Size | Type | Description and notes |
|:---:|:---:|:---:|:---:|:---:|
| Header | 0 | 6 Bytes | int | "GRAT.." as an ASCII-encoded string |
| Width | 6 | 4 Bytes | int | The horizontal dimensions of the map (maximum X coordinates) |
| Height | 10 | 4 Bytes | int | The vertical dimensions of the map (maximum Y coordinates) |
| Tiles | 14+ | 20 Bytes | struct | The given size is per tile (see below for details about the layout) |

## Tile Data

Each entry represents a block (tile) of the given map, and is structured as follows:

| Field | Offset | Size | Type | Description and notes |
|:---:|:---:|:---:|:---:|:---:|
| Altitude | 0 | 4 | float | Altitude at the bottom left corner, i.e., at (0, 0) relative coordinates |
| Altitude | 4 | 4 | float | Altitude at the bottom right corner, i.e., at (X, 0) relative coordinates |
| Altitude | 8 | 4 | float | Altitude at the top left corner, i.e., at (0, Y) relative coordinates |
| Altitude | 12 | 4 | float | Altitude at the top right corner, i.e., at (X, Y) relative coordinates |
| Block Type | 16 | 1 | byte | Used to determine walkable/snipable properties (see explanation below) |
| Unknown | 17 | 3 | int | (I haven't found any information about this field whatsoever) |

The corners of each tile are adjusted via the altitude parameters to realise many different slope variations.

I believe zero represents the engine's minimum height value (not the ground level as perceived ingame). There may also be some issues if setting them to negative values, or values that are too high for the client to render, but this needs further testing.

## Block Types

There are several types that determine whether characters can walk over the tile, or snipe through it using ranged attacks and spells. Unfortunately the interpretation of the type isn't straight-forward, as it differs depending on whether or not the tile is considered "underwater".

	Assumption: A tile is considered "underwater" if the (entire?) block is below the water level that was defined in the RSW file referring to the same map. I'd expect that the highest corner altitude must be below this level for the tile to count as being below water level, but I haven't confirmed this (yet).

**Edit: According to the OpenKore wiki, the AVERAGE depth must be HIGHER than the water level in order for a tile to be considered underwater. This seems highly counter-intuitive, but wouldn't surprise me really...**

This is the code snippet they gave to determine what I'd call the ``isUnderwaterTile`` property:

	float averageDepth = (gat_block.upperLeftHeight + gat_block.upperRightHeight
	+ gat_block.lowerLeftHeight + gat_block.lowerRightHeight) / 4;
	float waterLevel = readWaterLevelFromRSWFile();

	// The '>' is NOT a typo!
	if (averageDepth > waterLevel)
	// Block is below water level
	else
	// Block is above water level

The main difference between both types is that the default value of zero will cause ground tiles to be walkable, while underwater tiles are not usually considered walkable. This appears logical since actors can only cross very shallow water and the game doesn't support any notion of "swimming" or "diving".

### Ground-level tiles

The interpretation for ground tiles is mostly known:

| Value | Walkable | Snipable | Interpretation
|:---:|:---:|:---:|:---:|:---:|
| 0 | YES | ? | Regular walkable block |
| 1 | NO | ? | Regular non-walkable block |
| 2 | NO | NO | Non-walkable water (not snipable) |
| 3 | YES | ? | Shallow (walkable) water |
| 4 | NO | YES | Non-walkable water (snipable) |
| 5 | NO | YES | Cliff (snipable) |
| 6 | NO | NO | Cliff (not snipable) |
| 7 | ? | ? | Unknown  |

I'd assume that walkable tiles are always considered snipable, and non-walkable tiles can are only snipable if explicitly defined as such, but again this would need more testing to confirm.

### Underwater tiles

Interpreting underwater tiles is a bit less obvious, but also mostly understood:

| Value | Walkable | Snipable | Interpretation
|:---:|:---:|:---:|:---:|:---:|
| 0 | NO | ? | Not walkable |
| 1 | YES | ? | Walkable |
| 2 | NO | NO | Not walkable (and not snipable) |
| 3 | YES | ? | Walkable |
| 4 | ? | ? | Unknown |
| 5 | NO | YES | 5 - Not walkable (but snipable) |
| 6 | NO | NO | 6 - Not walkable (and not snipable) |
| 7 | ? | ? | Unknown |

## Notes

All offsets for the header are given in bytes from the start of the file. The offset for the tile data is relative to the beginning of the Tile Data entry in the GAT file.

Since the header size is fixed but the tile data is not, only the first tile will start at the basic offset. When calculating offsets for subsequent tiles, one needs to account for the size of all previous tile data that came before it and add it to the fixed offset

The header itself must be an ASCII-encoded representation of the string ``GRAT\001\002`` (meaning the hexadecimal bytes ``47 52 41 54 01 02``) in order to be considered valid by the client. This information is otherwise useless and can safely be discarded when working with the files outside of the game, but must be re-added if the goal is to create a map file that the client will read.

## TODO

* What about the different versions?
* Any way to determine what the unknown fields are used for?
* Tile order? Presumably they start at (0,0) and proceed in direction of the x axis?
* What is the unit of the altitude? RoBrowser multiplies the float value by 0.2, but I've no idea why
