# Format: Binary CSV tables

| | |
|---|---|
| Status | Lead (mostly Observed; three items not seen on the disc are marked Lead) |
| Byte order | Big-endian on GameCube |
| Where it lives | MST archive entries whose type is `csv`. See [mst-archive.md](mst-archive.md). |
| Provenance | Core provenance log, 2026-10-28 entries titled "Binary CSV …": [header](https://github.com/rusty-arms/core/blob/main/docs/provenance-log.md#2026-10-28-binary-csv-table-header), [row directory](https://github.com/rusty-arms/core/blob/main/docs/provenance-log.md#2026-10-28-binary-csv-row-directory), [keys](https://github.com/rusty-arms/core/blob/main/docs/provenance-log.md#2026-10-28-binary-csv-row-keys), [fields](https://github.com/rusty-arms/core/blob/main/docs/provenance-log.md#2026-10-28-binary-csv-field-records-and-types), [numbers](https://github.com/rusty-arms/core/blob/main/docs/provenance-log.md#2026-10-28-binary-csv-numbers), [strings](https://github.com/rusty-arms/core/blob/main/docs/provenance-log.md#2026-10-28-binary-csv-narrow-strings), [wide strings](https://github.com/rusty-arms/core/blob/main/docs/provenance-log.md#2026-10-28-binary-csv-wide-strings), [layout](https://github.com/rusty-arms/core/blob/main/docs/provenance-log.md#2026-10-28-binary-csv-layout-and-padding), [variation](https://github.com/rusty-arms/core/blob/main/docs/provenance-log.md#2026-10-28-binary-csv-variation-between-tables), [byte order](https://github.com/rusty-arms/core/blob/main/docs/provenance-log.md#2026-10-28-binary-csv-byte-order), [rebuild](https://github.com/rusty-arms/core/blob/main/docs/provenance-log.md#2026-10-28-binary-csv-byte-for-byte-rebuild) |
| Last updated | 2026-10-28, observer session in the lab |

## Summary

A binary CSV table is one MST entry of type `csv`. It holds a list of rows. Each row has a text key and a list of fields, and each field is a number, a string or a wide string. Nothing in the table fixes how many fields a row has or what type each one is: every field carries its own type. The layout is a 16-byte header, a directory of 16 bytes per row, and then each row's key, fields and strings in turn, with no gaps beyond a small pad after each key.

The only lead used was [DJs_MA_TOOLs](https://github.com/dj0wns/DJs_MA_TOOLs) (MIT, copyright 2015 Romet and later contributors), at commit `b74fc4b`: its `mst_loaders/csv.py` reader, its `lib/csv_classes.py` writer and its `csv_rebuilder.py`. It calls each row a "table" and each field a "value". No code was taken from it. Each fact below was checked against all 1291 `csv` entries in the owner's GameCube archive unless the section says **Lead**.

All integers are unsigned, in the table's byte order. Offsets are from the start of the table, that is from the start of the MST entry's data, not from the start of the archive.

## Header

The header is four 32-bit words (16 bytes). *Observed in all 1291 tables.*

| Offset | Size | Meaning |
|---|---|---|
| 0 | 4 | Table size: the table's length in bytes. It always equals the MST entry's length. |
| 4 | 4 | Row count. At least 1; 111 tables have exactly one row. |
| 8 | 4 | Row directory offset. Always 16, so the directory follows the header directly. |
| 12 | 4 | Unknown. Always zero. |

*Lead:* DJs_MA_TOOLs reads the word at 12 under a name suggesting a time, and writes it as "flags". It is zero in every table on the disc, so its meaning can't be checked here. A writer should write zero.

## Row directory

The directory has one 16-byte record per row, in row order, starting at offset 16. *Observed in all 6495 rows.*

| Offset | Size | Meaning |
|---|---|---|
| 0 | 4 | Key offset: where the row's key starts. |
| 4 | 4 | Key length in bytes, not counting the NUL after it. Never zero. |
| 8 | 2 | Field count. Never zero. |
| 10 | 2 | Row index: the row's position in the directory, counting from 0. It always equals that position. |
| 12 | 4 | Fields offset: where the row's field records start. |

The field count is 16 bits wide. The largest observed is 816 fields in one row.

## Row keys

*Observed in all 6495 rows:*

- A key is ASCII text followed by one NUL byte. Its bytes are all in the printable range `0x20`–`0x7E`, and none has a leading or trailing space.
- The longest key observed is 43 bytes.
- After the NUL come zero bytes up to the next multiple of 4, measured from the start of the table. The row's field records start there. So the fields offset is always the key offset plus the key length plus 1, rounded up to a multiple of 4.
- The key itself is **not** aligned. It starts wherever the previous row's data ended. See [Layout](#layout).
- Keys need not be unique. In 20 of the 1291 tables two or more rows share a key, and in each of those the duplicates are identical byte for byte, not just case-insensitively equal. A reader must keep every row in order, not a map from key to row.

How the game chooses between rows with the same key hasn't been observed.

## Field records

Each field record is three 32-bit words (12 bytes). A row's records are contiguous, starting at the row's fields offset. The first word is the field's type. *Observed in all 83741 fields.*

| Type | Meaning | Second word | Third word |
|---|---|---|---|
| 0 | String | String offset | Length in bytes, not counting the NUL |
| 1 | Number | 32-bit float | Always zero |
| 2 | Wide string | String offset | Length in UTF-16 code units, not counting the terminator |

The disc has 53903 numbers, 28512 strings and 1326 wide strings. No other type value occurs.

*Lead:* DJs_MA_TOOLs lists a fourth type, 3, named "count". It appears in no table on the disc, and the lead's reader rejects it, so what it would hold is unknown. A reader should reject unknown types.

### Numbers

*Observed in all 53903 numbers:*

- The second word is an IEEE 754 single-precision float in the table's byte order.
- Every one is finite, and none is subnormal. 1617 are negative, and 15 of those are negative zero, which a writer must keep as negative zero.
- 43802 hold whole values, and 10101 don't. Nothing in the record marks a number as an integer.

*Observed, for converters:* the lead's text export rounds numbers to five decimal places. That changes the stored bits of 3411 of the 53903 numbers. A writer that wants to reproduce a table must keep each number's exact 32 bits.

### Strings

*Observed in all 28512 strings:*

- The string offset points at the first byte. The length counts bytes. One NUL byte follows the text.
- Every byte is printable ASCII, `0x20`–`0x7E`: no control characters and nothing above `0x7E`. The longest is 52 bytes.
- No string is empty.
- A string has no alignment. It starts at whatever offset the data before it ended on.

### Wide strings

*Observed in all 1326 wide strings:*

- The text is UTF-16 in the table's byte order: big-endian on GameCube. The length counts 16-bit code units. Two zero bytes, one zero code unit, follow the text.
- Every code unit is in `0x20`–`0x7E` or is `0x0A`, a line feed. Line feeds occur in 45 wide strings, and no other control character occurs anywhere. No code unit is above `0x7E`, so no surrogate pairs occur. The longest has 142 code units.
- One wide string is empty: its length is zero, and it is just the two-byte terminator.
- A wide string is **not** aligned to 2 bytes. 126 of the 1326 start at an odd offset, because the data before them ended on one. A reader must not assume 16-bit alignment.

Which fields use wide strings: 89 tables hold at least one, and in 5 tables every row has one. Nothing in the header or directory flags this. The type is per field.

## Layout

Everything after the directory is packed in row order with nothing between items except the pad after each key. *Observed in all 1291 tables:*

1. Header, 16 bytes.
2. Row directory, 16 bytes per row.
3. For each row, in directory order:
   1. its key and NUL;
   2. zero padding to a multiple of 4, counted from the start of the table;
   3. its field records, 12 bytes each;
   4. the text of its string and wide-string fields, in field order. Each one starts exactly where the previous one ended, terminator included. Numbers have no text here.
4. The table ends exactly at the end of the last row's last string, or at the end of its last field record if that row has no strings. There is no padding at the end, so a table's length needn't be a multiple of 4. 582 of the 1291 aren't. (The MST archive pads between entries; see [mst-archive.md](mst-archive.md).)

So the first row's key starts at 16 + 16 × the row count. Each later row's key starts at the end of the row before it. No two fields share a string: each field's text is stored once, in its own place.

A reader should still follow the offsets rather than assume this order. A writer that follows it reproduces every table on the disc. See [Writing a table](#writing-a-table).

## Variation between tables

There is no per-table schema. *Observed:*

- Rows in one table can have different field counts. Only 217 of the 1291 tables have one field count for every row.
- The type at a given field position can change from row to row. Only 199 tables have the same sequence of field types in every row. Of the other 1092, 935 have at least one field position whose type differs between rows.

A reader must therefore read each field's type from its record. A text converter must store each field's type rather than infer it from the text or from other rows.

## Byte order

*Observed:* on GameCube every word, halfword, float and wide-string code unit is big-endian, the same as the archive it sits in.

*Lead:* DJs_MA_TOOLs treats Xbox tables as little-endian throughout, wide strings included. That can't be checked with the GameCube disc. As with the MST archive, the lead's command-line help gets the GameCube byte order the wrong way round, and its code gets it right. A reader can take the byte order from the archive's magic; see [mst-archive.md](mst-archive.md#magic-and-byte-order).

## Writing a table

Given its rows in order, each row a key and a list of typed fields, a writer reproduces a table byte for byte like this. *Observed:* this procedure, written independently in the lab, rebuilt all 1291 tables on the disc exactly from their decoded rows.

1. Write the header: the table size (fill it in at the end), the row count, 16, and 0.
2. Reserve 16 bytes per row for the directory.
3. For each row, at the current position:
   1. write the key and a NUL;
   2. pad with zeros to a multiple of 4 from the start of the table;
   3. write the row's field records, with each string's offset set to where its text will go: right after the row's field records, each string after the one before;
   4. write each string's text with its terminator, one NUL byte for a string and two zero bytes for a wide string, in field order;
   5. fill in the row's directory record: key offset, key length, field count, the row's index and the fields offset.
4. Set the table size to the final length. Add no padding at the end.

## Example

*Made up* to show the layout. It is not taken from the disc. A big-endian table with two rows: row 0 has key `speed` and two fields, the number 2.5 and the string `fast`; row 1 has key `label` and one field, the wide string `Hi`.

| Offset | What | Value |
|---|---|---|
| 0 | Table size | 110 |
| 4 | Row count | 2 |
| 8 | Row directory offset | 16 |
| 12 | Unknown word | 0 |
| 16 | Directory, row 0 | key offset 48, key length 5, field count 2, index 0, fields offset 56 |
| 32 | Directory, row 1 | key offset 85, key length 5, field count 1, index 1, fields offset 92 |
| 48 | Row 0 key | `speed`, NUL |
| 54 | Padding | 2 zero bytes, up to 56 |
| 56 | Row 0, field 0 | type 1, float 2.5 (`0x40200000`), 0 |
| 68 | Row 0, field 1 | type 0, offset 80, length 4 |
| 80 | Row 0, string text | `fast`, NUL |
| 85 | Row 1 key | `label`, NUL (unaligned: it follows the string directly) |
| 91 | Padding | 1 zero byte, up to 92 |
| 92 | Row 1, field 0 | type 2, offset 104, length 2 |
| 104 | Row 1, wide string text | `H`, `i` as big-endian UTF-16 (4 bytes), then 2 zero bytes, ending at 110 = the table size |

## How to check it

An implementer can build synthetic tables like the example and check that a reader:

- takes every count and offset from the header and directory, not from assumed positions;
- reads a field's type from each record, with rows of different lengths and mixed types in one column position;
- reads wide strings that start at an odd offset;
- keeps rows with duplicate keys, in order;
- keeps negative zero and the exact bits of every number.

A writer should rebuild each synthetic table byte for byte. On the owner's disc, every `csv` entry should decode and rebuild to the same bytes, and its table size should equal the entry's length.

## Open questions

- What the header word at offset 12 means. It is zero in every table. The lead calls it flags.
- Whether type 3 exists, and what it holds. Only the lead mentions it.
- Whether the game reads the table size, the row directory offset or the row index fields, or trusts the layout. An observer could change one in a copy of the archive and watch in Dolphin.
- How the game looks up rows: by key or by position, case-sensitively or not, and which row it uses when keys repeat.
- How the game uses each table: which fields it reads as numbers, strings or wide strings, and what happens if a field has a type it doesn't expect. That belongs in behaviour specs for the systems that read each table.
- Byte order and layout in the Xbox and PS2 versions. Only the GameCube disc has been observed.
