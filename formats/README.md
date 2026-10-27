# Format specs

One spec per format, written from the owner's disc and allowed public docs. Start from [the format template](../templates/format.md). Rules: [CONTRIBUTING.md](../CONTRIBUTING.md).

The planned list comes from the research brief's public knowledge (core's `docs/research/brief.md`, section 6). Only the MST archive has been observed so far.

| Format | Spec | Status | Public leads |
|---|---|---|---|
| Disc layout: which files hold what | None yet | Not started | Brief: one archive holds almost everything; movies and streamed audio are separate |
| MST archive: header, entry table, per-type compiler versions | [mst-archive.md](mst-archive.md) | Lead (mostly Observed) | DJs_MA_TOOLs (MIT) reads and writes it |
| Textures | None yet | Not started | None public; GX texture formats are documented |
| Meshes | None yet | Not started | None public |
| Worlds | None yet | Not started | DJs_MA_TOOLs edits them in part |
| Animations | None yet | Not started | None public |
| Binary CSV tables | None yet | Not started | DJs_MA_TOOLs converts them to and from JSON |
| Fonts | None yet | Not started | None public |
| AMX bytecode scripts | None yet | Not started | DJs_MA_TOOLs has an experimental disassembler; the scripts are written in Small, now called Pawn |
| AI node graphs | None yet | Not started | None public |
| Audio banks and streamed audio | None yet | Not started | DSP ADPCM is documented |
| Movies | None yet | Not started | Bink, decoded by FFmpeg's open decoder |
