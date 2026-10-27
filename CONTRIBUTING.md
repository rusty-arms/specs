# Writing specs

A spec says what a format looks like or how the game behaves, precisely enough that someone who has never seen the game could implement it. It is written by an observer, in their own words.

## What a spec may contain

- Descriptions of structure and behaviour in your own words: field order, sizes, byte order, meaning, valid ranges, what happens when.
- Magic numbers and other bytes that only identify a structure. Nothing beyond that from the disc.
- Synthetic examples: byte layouts and values you made up to illustrate a structure. Say that they are made up.
- Facts from allowed public sources, cited by name: DJs_MA_TOOLs (MIT), public GameCube hardware and media documentation, public gameplay descriptions.
- Facts from Exavolt's public docs, only as leads, marked as such until an observer confirms them.

## What a spec must never contain

- Code, pseudo-code or comments copied or paraphrased from the original game, from decompiler output, or from any forbidden source in the [clean-room policy](https://github.com/rusty-arms/core/blob/main/docs/clean-room.md).
- Retail bytes beyond identifying magic: no hex dumps, tables, strings, scripts or other data copied from the disc.
- Screenshots, frame dumps, recordings or any other capture of the game. Describe what you saw instead.
- Paths into the lab, or anything copied from it.
- Names of, or links to, forbidden material. Describe it generically if you must mention it.

If forbidden material turns up while you work: stop, don't summarise or quote it, tell the owner, and log an incident in core's provenance log.

## Provenance

Every fact in a spec needs a dated entry in core's [provenance log](https://github.com/rusty-arms/core/blob/main/docs/provenance-log.md): what was learned, how, and by which session. The spec's header links to those entries. A fact without an entry doesn't go in.

## Status

Each spec has one status, shown in its header and in its folder's index:

| Status | Meaning |
|---|---|
| Lead | Taken from public docs. Not yet checked against the disc or in Dolphin. |
| Observed | An observer has confirmed it against the owner's disc or in Dolphin. |
| Round-trips | A parser or behaviour built from the spec matches the disc locally, and its synthetic fixtures pass in CI. |

A spec may mix statuses section by section; the header shows the lowest.

## Workflow

1. Copy a template from [`templates/`](templates/) into `formats/` or `behaviour/`. Name the file after the format or behaviour in lower case with hyphens, such as `formats/archive.md`.
2. Write the spec, and add the provenance entries to core in the same session.
3. Add a row to the folder's index.
4. Open a pull request labelled `spec`. Changes to how the wall works also get `clean-room`.

Implementers who find a spec unclear or wrong open an issue labelled `spec`. They don't edit the spec themselves when the fix needs the lab.
