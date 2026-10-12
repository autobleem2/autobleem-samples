# autobleem-samples

The sample games [AutoBleem](https://github.com/autobleem2/autobleem)'s installer puts on a fresh stick,
so the shelf is not empty on the first start: homebrew whose licence allows redistribution, listed in
`tools/samples/samples.json`. This repository builds the *sample pack*, a `samples-<date>.tar.gz`; it does
not contain any game file itself.

## Who uses it

The pack is fetched by the launcher's `payload_linux/install.sh` from the download site's
`samples/latest.json` (`--no-samples` skips it) and staged by autobleem-appliance into its package; see
`CLAUDE.md` ("Consumers"). This repository has no tests
of its own: the CI build itself is the check, because it fails when an upstream file moved or changed.

## Games today

| Game | System | Licence |
|---|---|---|
| Nova the Squirrel | NES | GPL-3.0 |
| Asteroids | SNES | MIT |
| Castle Platformer | SNES | MIT |
| Alex vs Bus - The Race | Mega Drive | GPL-3.0 (code), CC BY-SA 4.0 (assets) |

There is no PS1 game at the moment (the manifest's `_comment` says why); the PS1 code path in
`build_samples.py` is kept for the next one. Each game keeps its own licence, linked from its manifest
entry and reproduced in the pack's `SAMPLES.md`. `LICENSE` (GPL-3.0-or-later) covers the tooling only.

## Layout

```
tools/build_samples.py        downloads, checks and packs the games; its docstring describes the pack tree
tools/samples/samples.json    the manifest, one entry per game under "games"
tools/samples/covers/         cover PNGs, named by each entry's "cover" (checked in)
tools/samples/shots/          screenshots named by each entry's "shot", input for make_covers.py only
tools/samples/make_covers.py  draws a cover from the screenshot (needs Pillow)
.github/workflows/build.yml   CI
```

`covers/tetrade.png` and `shots/tetrade_title.png` belong to the removed PS1 game and are not used.

## Building

```
python3 tools/build_samples.py [--out DIR] [--cache DIR] [--date YYYYMMDD]
```

Standard library only. Each file is downloaded from its upstream URL and checked against the entry's
sha256 (a mismatch aborts). The result is `<out>/samples-<date>.tar.gz` and `<out>/samples-<date>.json`
(the site page and `install.sh` read the json). The tarball holds `RetroArch/roms/<db name>/`,
`RetroArch/thumbnails/<db name>/Named_Boxarts/` and `SAMPLES.md`; PS1 games would go in `Games/<title>/`.
The PS1 path reads `src/resources/pcsx.cfg` and `make_covers.py` reads a font from `payload/Themes/`;
neither exists in this repository, so both need fixing before a PS1 game is added.

## Publishing

`build.yml` (runs only when the repository variable `AB_CI_ENABLED` is `true`) builds the pack on every push
and PR to `develop`/`master`. A `v*` tag creates a GitHub release with the tarball and json. A push to
`develop` updates the rolling `nightly` pre-release (tarball only). A `v*` tag, or a manual run with
`publish` on, also pushes the pack to the download site's `samples/` on the self-hosted runner.

## Adding a sample

1. Confirm the licence allows redistribution; it ends up in `SAMPLES.md`.
2. Add an entry to `samples.json` with `id`, `system` (`nes`, `snes`, `md`, or `psx`), `title`, `author`,
   `year`, `players`, `licence`, `licence_url`, `source`, `description`, `files` (`url`, `sha256`, `name`,
   optional zip `member`), `cover` and `shot` (empty if none). A new ROM system also needs a line in
   `SYSTEM_DB` and in the name map in `samples_md()` in `build_samples.py`.
3. Run `python3 tools/samples/make_covers.py`, commit the PNG in `covers/` and the screenshot in `shots/`.
4. Run `build_samples.py` and look at the tarball. Keep `.py`/`.json`/`.md` files LF (`.gitattributes`).
