# CLAUDE.md: specs

## What this repo is

`specs` holds the Rusty Arms format and behaviour specs. It is the clean-room wall between observers, who look at the game, and implementers, who build the engine. See [README.md](README.md) and [CONTRIBUTING.md](CONTRIBUTING.md).

It holds **no code from the original game and no game data**, and it never will. Project policy, decisions and status live in `core`: `docs/clean-room.md`, `docs/provenance-log.md`, `PROGRESS.md`.

## Which role you are

- **Observer sessions** start inside the lab. They may write and update specs here, in their own words, and add a provenance entry in core for every fact.
- **Every other session is an implementer, or is maintaining this repo.** It may fix wording, links and layout, but must not add a fact that has no provenance entry. It never opens the lab. Questions that need the lab go in an issue labelled `spec`.

## Clean-room rules (short form)

- **Never** access, search for, fetch, clone or read: the 2019 source leak or anything built on it; the community asset compiler; archive copies of the source or prototype discs; the community HD texture pack; decompiler output of the retail executable.
- **Never** read anything in the lab (`~/Develop/rusty-arms/ma-lab`) unless this session started inside it as an observer.
- Exavolt's docs are leads only, never copy its code. DJs_MA_TOOLs is MIT and may be cited with attribution.
- If forbidden material appears anyway: stop, don't summarise or quote it, tell the owner, and log an incident in core's `docs/provenance-log.md`.
- **Never commit** retail bytes beyond identifying magic, disc images, extracted files, screenshots or captures of the game, anything from the lab, or blocklist terms.
- In committed files, describe forbidden material generically. Never name the owner; write "the owner".

## Session discipline

1. Start by reading core's `PROGRESS.md` and this repo's `git log -10`.
2. One task per session.
3. Make small commits, one per step.
4. Record where you stopped in core's `PROGRESS.md` before ending the session.

## Approval and guard rails

- Changes to `.github/` need the owner's explicit approval in the chat. CODEOWNERS enforces this on GitHub as well.
- **Never weaken or edit** `scripts/check_no_retail.py`, `scripts/check_links.py`, `.githooks/` or `.github/workflows/guard.yml`. They are copies of core's: change them in core first, then copy them here unchanged.
- Do not route around a blocked tool call. A block is the answer; report it.

## Checks

```sh
python3 scripts/check_no_retail.py --all
python3 scripts/check_links.py
yamllint -s .
```
