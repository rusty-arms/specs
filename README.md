# specs

Format and behaviour specs for Rusty Arms, a clean-room engine that runs a 2003 GameCube game from the player's own disc. Observers write these specs in their own words. Implementers build the engine and tools from them.

**This repo is the clean-room wall.** It holds no code and no data from the original game, and it never will. Project policy lives in core: [clean-room policy](https://github.com/rusty-arms/core/blob/main/docs/clean-room.md), [provenance log](https://github.com/rusty-arms/core/blob/main/docs/provenance-log.md) and [project status](https://github.com/rusty-arms/core/blob/main/PROGRESS.md).

## Who does what

| Role | May use | Does here |
|---|---|---|
| Observer | The owner's disc, Dolphin, the local lab, allowed public docs | Writes and updates specs, and logs each fact in core's provenance log |
| Implementer | These specs, allowed public docs, synthetic fixtures | Reads specs, and opens an issue labelled `spec` when a spec is unclear or wrong |

Implementers never see the lab. If a spec needs a fact that only the lab can give, the question goes to an observer as an issue.

## Layout

| Folder | What goes in it |
|---|---|
| [`formats/`](formats/README.md) | How the disc's files and archive entries are laid out: one spec per format |
| [`behaviour/`](behaviour/README.md) | How the game behaves when it runs: timing, rules, per-level logic |
| [`templates/`](templates/) | Starting points for a new spec |

How to write a spec, and what may and may not go in one: [CONTRIBUTING.md](CONTRIBUTING.md).

## Status

No spec has been written yet. Each folder's index lists the planned specs and their state.

## Develop

```sh
python3 scripts/check_no_retail.py --all
python3 scripts/check_links.py
yamllint -s .
```

CI runs all three on every push and pull request. `.githooks/pre-commit` runs the retail-data guard and the link check; enable it with `git config core.hooksPath .githooks`.

Agent sessions get the clean-room guard and the session rules from the [agent-kit](https://github.com/rusty-arms/agent-kit) plugin, which `.claude/settings.json` enables.

## Licence

MIT OR Apache-2.0, at your option: [LICENSE-MIT](LICENSE-MIT), [LICENSE-APACHE](LICENSE-APACHE).
