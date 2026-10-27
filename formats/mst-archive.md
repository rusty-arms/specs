# Format: MST archive

| | |
|---|---|
| Status | Lead (most sections Observed; see each section) |
| Byte order | Big-endian on GameCube, detected from the magic |
| Where it lives | The disc's one `.mst` file, in the root of the disc's file system. It holds almost all of the game's compiled data. |
| Provenance | Core provenance log, 2026-10-27 entries titled "MST …": [magic](https://github.com/rusty-arms/core/blob/main/docs/provenance-log.md#2026-10-27-mst-magic-and-byte-order), [version word](https://github.com/rusty-arms/core/blob/main/docs/provenance-log.md#2026-10-27-mst-version-word), [size](https://github.com/rusty-arms/core/blob/main/docs/provenance-log.md#2026-10-27-mst-archive-size-field), [entry table](https://github.com/rusty-arms/core/blob/main/docs/provenance-log.md#2026-10-27-mst-entry-table-position-and-capacity), [support table](https://github.com/rusty-arms/core/blob/main/docs/provenance-log.md#2026-10-27-mst-support-table), [data offset](https://github.com/rusty-arms/core/blob/main/docs/provenance-log.md#2026-10-27-mst-data-offset), [type version words](https://github.com/rusty-arms/core/blob/main/docs/provenance-log.md#2026-10-27-mst-per-type-version-words), [entry row](https://github.com/rusty-arms/core/blob/main/docs/provenance-log.md#2026-10-27-mst-entry-row-layout), [names](https://github.com/rusty-arms/core/blob/main/docs/provenance-log.md#2026-10-27-mst-entry-names-and-types), [placement](https://github.com/rusty-arms/core/blob/main/docs/provenance-log.md#2026-10-27-mst-entry-data-placement), [timestamps](https://github.com/rusty-arms/core/blob/main/docs/provenance-log.md#2026-10-27-mst-timestamps), [content word](https://github.com/rusty-arms/core/blob/main/docs/provenance-log.md#2026-10-27-mst-entry-content-word), [older name width](https://github.com/rusty-arms/core/blob/main/docs/provenance-log.md#2026-10-27-mst-older-16-byte-names-lead) |
| Last updated | 2026-10-27, observer session in the lab |

## Summary

An MST archive is a flat container: a fixed 108-byte header, then a table of named entries, then a second table of names with no data (the "support" table), then the entries' data. Each entry row gives the entry's name, the absolute offset and exact length of its data, a timestamp and a word derived from its content. The extension in an entry's name says which loader reads it. There are no directories and no compression at this level.

The only lead used was [DJs_MA_TOOLs](https://github.com/dj0wns/DJs_MA_TOOLs) (MIT, copyright 2015 Romet and later contributors), at commit `b74fc4b`: its `mst_lib/mst.py` reader and `mst_insert.py` writer. No code was taken from it. Each fact below was checked against the owner's disc unless the section says **Lead**.

All integers are unsigned 32-bit in the archive's byte order unless stated otherwise. Offsets are from the start of the archive file.

## Header

The header is 27 words (108 bytes). The entry table starts right after it, at offset 108. *Observed.*

| Offset | Size | Meaning |
|---|---|---|
| 0 | 4 | Magic. See [Magic and byte order](#magic-and-byte-order). |
| 4 | 4 | Version word. See [Version word](#version-word). |
| 8 | 4 | Archive size: the whole file's length in bytes. |
| 12 | 4 | Entry count: used rows in the entry table. |
| 16 | 4 | Free entry rows: unused rows that follow the used ones. |
| 20 | 4 | Support count: used rows in the support table. |
| 24 | 4 | Free support rows: unused rows that follow those. |
| 28 | 4 | Data offset: where the entries' data begins. |
| 32 | 40 | Ten per-type version words. See [Per-type version words](#per-type-version-words). |
| 72 | 4 | Unknown. A small non-zero integer on the observed disc. |
| 76 | 32 | Eight words, all zero on the observed disc. Treat as reserved. |

*Lead, partly contradicted:* the DJs_MA_TOOLs writer lists only 26 header words: up to the per-type words, then eight reserved ones. Its reader and its writer both start the entry table at 108, which is 27 words. The disc agrees with 108.

On the observed disc the archive size equals the file's length exactly. A reader can use it as a sanity check, and a writer must update it.

### Magic and byte order

The magic is the 32-bit value `0x474E4146`. Read in the archive's own byte order it always gives that value, so its four bytes spell `GNAF` in a big-endian archive and `FANG` in a little-endian one. A reader picks the byte order that makes the magic come out right, and uses it for every other field.

- *Observed:* the GameCube archive starts with `GNAF` and is big-endian throughout.
- *Lead:* DJs_MA_TOOLs treats `FANG` as little-endian and says it belongs to the Xbox and PS2 versions. Not checked: only the GameCube disc is available. The DJs_MA_TOOLs README's command-line help says the opposite about GameCube byte order. The disc agrees with its code, not its help text.

### Version word

The version word holds four 8-bit fields, from most significant to least:

| Bits | Meaning |
|---|---|
| 31–24 | Platform flags |
| 23–16 | Major version |
| 15–8 | Minor version |
| 7–0 | Sub-version |

- *Observed:* the GameCube archive is version 1.8.0, and its platform flags are `0x18`: bits `0x10` and `0x08` set.
- *Lead:* DJs_MA_TOOLs reads flag `0x10` as GameCube, `0x01` as Xbox, `0x20` as PS2 and `0x04` as PC. The GameCube disc is consistent with `0x10`, but the other flags can't be checked here.
- *Unknown:* what flag `0x08` means.

The version matters for the name width in the entry table; see [Entry name width by version](#entry-name-width-by-version).

### Per-type version words

The ten words at offsets 32 to 71 are small non-zero integers on the observed disc.

*Lead:* DJs_MA_TOOLs names them, in order, as the compiler versions for these entry types: textures (`tga`), meshes (`ape`), animations (`mtx`), tables (`csv`), fonts (`fnt`), scripts (`sma`), `gt`, wave banks (`wvb`), `fpr` and cameras (`cam`). The archive itself doesn't confirm this. No value appears as a constant field near the start of every entry of the matching type, and the archive has no `wvb` entries.

A reader should keep these words and pass them to the type loaders, but should not depend on them until the per-type specs confirm their meaning.

## Entry table

The entry table starts at offset 108. It has *entry count + free entry rows* rows of 36 bytes each. The first *entry count* rows are in use. Every unused row after them is all zero bytes. *Observed.*

Rows are in no particular order: not by name, not by data offset, not by timestamp. A reader finds an entry by name. *Observed.*

### Entry row

| Offset | Size | Meaning |
|---|---|---|
| 0 | 20 | Name: ASCII, NUL-terminated. See [Entry names](#entry-names). |
| 20 | 4 | Location: absolute offset of the entry's data in the archive. |
| 24 | 4 | Length: exact length of the data in bytes. It is not rounded up, and isn't always a multiple of 4. |
| 28 | 4 | Timestamp. See [Timestamps](#timestamps). |
| 32 | 4 | Content word. See [Content word](#content-word). |

*Observed*, for the GameCube archive's version 1.8.0. Every used row's location and length point inside the file. No two entries' data overlap, and no two entries share a location.

### Entry name width by version

*Lead only:* DJs_MA_TOOLs reads the name as 16 bytes, making a 32-byte row, in archives whose major version is 1 and minor version is 7 or less. It reads 24 bytes for PS2 archives and 20 bytes otherwise. Only the 20-byte form has been observed.

### Entry names

*Observed:*

- Read the name up to the first NUL. **Ignore the bytes after it.** In about one row in twenty they are not zero, and they hold leftover bytes, not a second string.
- The longest name observed has 15 characters plus its NUL. A writer should keep names to 19 characters at most, so the NUL still fits.
- There are no directories: no name contains a slash or a backslash.
- Names are lower case and unique, even when compared case-insensitively.
- Names use only letters, digits, space, `$`, `-`, `_` and exactly one `.`.
- The part after the dot is the entry's type, two or three letters long. These types occur in the GameCube archive: `tga`, `ape`, `mtx`, `csv`, `fnt`, `sma`, `gt`, `fpr`, `cam`, `wld`, `rdg` and `sfb`. Separate specs will cover what each holds.

How the game itself looks names up (case handling, hashing, a linear search) hasn't been observed. A reader that matches names case-insensitively will find every entry.

## Support table

The support table follows the entry table directly. It starts at 108 + 36 × (entry count + free entry rows), and it has *support count + free support rows* rows of 20 bytes each. The first *support count* rows are in use, and the rest are all zero. *Observed.*

| Offset | Size | Meaning |
|---|---|---|
| 0 | 16 | Name: ASCII, NUL-terminated, with only zero bytes after the NUL. |
| 16 | 4 | Timestamp, in the same form as an entry's. |

*Observed:*
- Support rows have no location and no length, and the archive holds no data for them.
- Their names follow the same rules as entry names. They all share one extension, which no entry in the main table uses, and none of them matches an entry name.

*Unknown:* what the support table is for. The name "support" comes from DJs_MA_TOOLs. The rows look like a record of source files that went into the build, but that hasn't been checked. A reader can skip the table, but a writer must keep it.

*Observed, possibly a coincidence:* on the GameCube disc the two tables have the same capacity, so entry count + free entry rows = support count + free support rows.

## Data region

The data offset in the header is the end of the support table, rounded up to the next multiple of 2048. The bytes between the end of the support table and the data offset are zero. The first entry's data, in file order, starts exactly at the data offset. *Observed.*

### Entry data placement

*Observed:*

- Every entry's data starts at a multiple of 32.
- Taken in file order, each entry starts at the end of the previous entry's data rounded up to the next multiple of 32, or else rounded up to the next multiple of 2048. Nothing else happens.
- Padding bytes between entries are zero.
- The 2048 rounding is used almost only before textures (`tga`), but not before all of them. What decides it is unknown; it lines up with a field in the texture's own header, which the texture spec will cover.
- The order of data in the file is not the table order, and entries of different types are interleaved. Why the data is in this order is unknown.
- The file ends at the last entry's data rounded up to a multiple of 32, with zero padding. That rounded end is the archive size in the header.

A reader must use each row's location and length and assume nothing about packing. A writer that rounds every entry up to 32, puts the data offset on a 2048 boundary and updates the size field produces an archive with the same structure. Whether the game needs the 2048 rounding for some textures is unknown.

## Timestamps

Entry and support timestamps are Unix times: seconds since 1970-01-01. On the observed disc every one falls between late 2000 and August 2003, which fits the game's development. None is zero. The time zone and what event the stamp records, such as source edit or compile, are unknown. *Observed.*

## Content word

The last word of an entry row depends only on the entry's content. *Observed:*

- It is zero for every entry of types `rdg`, `gt`, `cam` and `fnt`, and non-zero for every entry of the other types.
- Entries with byte-identical data always have the same word.
- Every non-zero value shared by two or more entries belongs to entries with identical data.

*Unknown:* the algorithm. It is not the standard CRC-32 (zlib), CRC-32/BZIP2, CRC-32/MPEG-2, CRC-32C, Adler-32, a byte sum, a 32-bit word sum or XOR, or FNV-1 or FNV-1a over the entry's data. It may be computed over the source asset rather than the compiled data.

*Lead:* DJs_MA_TOOLs calls this word a CRC in its writer and "unknown" in its reader. Its writer fills new rows with a running index rather than a checksum, yet its README presents that writer as the way to put modified files back into the game. If that works, the game doesn't check this word for new entries, but it hasn't been observed.

## Example

*Made up* to show the layout. It is not taken from the disc. A big-endian archive with two entries, one free entry row, no support rows and one free support row:

| Offset | What | Value |
|---|---|---|
| 0 | Magic | `GNAF` |
| 4 | Version word | flags `0x18`, version 1.8.0 |
| 8 | Archive size | 2176 |
| 12 | Entry count | 2 |
| 16 | Free entry rows | 1 |
| 20 | Support count | 0 |
| 24 | Free support rows | 1 |
| 28 | Data offset | 2048 |
| 32–107 | Per-type version words, unknown word and reserved words | any; zero here |
| 108 | Entry row 0 | name `alpha.csv`, location 2048, length 70, timestamp 1000000000, content word 0x12345678 |
| 144 | Entry row 1 | name `beta.tga`, location 2144, length 32, timestamp 1000000000, content word 0x9ABCDEF0 |
| 180 | Free entry row | 36 zero bytes |
| 216 | Free support row | 20 zero bytes |
| 236 | Padding | zero bytes up to 2048 |
| 2048 | `alpha.csv` data | 70 bytes |
| 2118 | Padding | 26 zero bytes, up to 2144 (a multiple of 32) |
| 2144 | `beta.tga` data | 32 bytes, ending at 2176 = the archive size |

## How to check it

An implementer can build synthetic archives like the example and check that a reader:

- detects the byte order from the magic;
- stops names at the first NUL even when the bytes after it are not zero;
- skips the free rows and the support table;
- finds entries by name whatever their table order.

On the owner's disc, a reader should find that every used row's data lies inside the file, that no two entries overlap, and that the archive size equals the file's length.

## Open questions

- The algorithm behind the content word, and whether the game checks it. An observer could find out by changing one entry's content word in a copy of the archive and watching in Dolphin whether that entry still loads.
- The meaning of platform flag `0x08`.
- What the support table records, and whether the game reads it.
- What the ten per-type version words mean, and which type each belongs to. The per-type specs may confirm the DJs_MA_TOOLs mapping.
- What the header word at offset 72 means.
- What decides 2048-byte placement for some textures, and whether the game needs it. The texture spec should settle the first part.
- How the game looks names up: case sensitivity, and any hashing or ordering it depends on.
- Byte order and name width in other versions of the archive. Only the GameCube archive's version 1.8.0 has been observed.
