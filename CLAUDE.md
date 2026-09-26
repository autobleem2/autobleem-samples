# autobleem-samples - developer context

The source of the sample games [AutoBleem](https://github.com/autobleem2/autobleem)'s installer puts on a
fresh stick (autobleem-main's `docs/decisions.md`: "Themes and samples have their own source
repositories", the same split as `autobleem-themes`). A homebrew pack, kept small on purpose (~1 MB
unpacked): a PS1 game or two and a handful of ROMs for the systems RetroArch ships cores for, each
redistributable under its own licence.

## Layout

```
tools/build_samples.py        packs the manifest below into the release archive - see its own docstring
tools/samples/samples.json    the manifest: one entry per game (id, system, title, author, licence,
                               licence_url, source, description, files[{url, sha256, name, member?}],
                               plus psx-only players/publisher/year/skip_boot_logo and rom-only system)
tools/samples/covers/         each game's cover art (PNG), by the manifest's "cover" field
tools/samples/shots/          reference screenshots (not shipped in the pack; for `make_covers.py`/review)
tools/samples/make_covers.py  draws a placeholder cover from a screenshot when a game has no box art
LICENSE                       GPL-3.0-or-later, for the tooling only - see "Licence" below
```

Nothing here is a game's own file: `build_samples.py` downloads every game from its upstream release URL,
sha256-checks it against the manifest, and stages it exactly as it will sit on a stick - see the script's
docstring for the tree inside the pack (`Games/<title>/`, `RetroArch/roms/<system>/`,
`RetroArch/thumbnails/<db>/Named_Boxarts/`, `SAMPLES.md`) and the two conventions it writes into a PS1
game's `Game.ini` (`Automation=0`, locked, so the scanner keeps the title/publisher/year/players written
here) and `pcsx.cfg` (`SlowBoot = 0` when the manifest's `skip_boot_logo` says the game's own boot logo
does not get past pcsx-ab's BIOS shell).

## Adding or updating a sample

Add an entry to `tools/samples/samples.json` (a `files[]` URL + its sha256, checked at build time - a
moved or changed upstream file fails the build loudly rather than shipping silently) and a cover under
`tools/samples/covers/`. Confirm the game's licence actually allows redistribution before adding it - the
description/licence/licence_url/source fields all end up in the pack's own `SAMPLES.md`, which is the
licence notice a player sees. `.json`/`.py` files are LF (`.gitattributes` enforces this - do not let a
Windows editor turn them back to CRLF); cover PNGs are committed as binary.

## Packaging and release

`python3 tools/build_samples.py --out DIR` writes `DIR/samples-<date>.tar.gz` + `DIR/samples-<date>.json`
(sha256, size, unpacked size, the games list - what the download site's page and `install.sh`'s "here's
what you got" summary read).

`.github/workflows/build.yml`, gated by the repository variable `AB_CI_ENABLED` like every autobleem2
repository: builds the pack on every push and pull request (so a moved or changed upstream file is caught
here, not by an installer); a `v*` tag publishes a GitHub release with the tarball + json attached, and
every push to `develop` replaces the rolling `nightly` pre-release's asset
(`autobleem2/autobleem-build`'s `nightly-release` action - the same scheme `autobleem-themes`, `proc_unzip`
and `ext_store` use). A separate `publish` job (kept from before the release/nightly jobs existed, pending
a decision on whether it stays alongside them) pushes the pack straight to the download site's `samples/`
on a `v*` tag or a manual dispatch with `publish` on, via `autobleem-repo`'s `repo_publish.sh` on the
self-hosted runner. No build image is needed: it is pure standard-library Python, nothing to compile.

## Consumers

The launcher's own `tools/build_samples.py`/`tools/samples/` copy is gone (D6): this repository is now the
sample pack's source. `payload_linux/install.sh` fetches the already-built pack from the download site's
`samples/latest.json` (`--no-samples` skips it) - that fetch is generic and already decoupled from where
the tooling that built the pack lives, so nothing there had to change beyond one comment. autobleem-appliance
takes a samples release the way it takes any other component's, staged into the package it assembles - see
autobleem-main's `docs/todo.md` D6.

## Licence

GPL-3.0-or-later (`LICENSE`) for `tools/build_samples.py` and `tools/samples/*.py`, matching the launcher.
Each sample game keeps its own licence (named in `tools/samples/samples.json` and reproduced in the pack's
`SAMPLES.md`) - this repository's licence does not extend to the games it packages.
