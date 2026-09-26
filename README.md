# autobleem-samples

The sample games [AutoBleem](https://github.com/autobleem2/autobleem)'s installer puts on a fresh stick,
so the shelf is not empty on the first start: homebrew whose licence allows redistribution, listed in
`tools/samples/samples.json`.

## Building the pack

```
python3 tools/build_samples.py [--out DIR] [--cache DIR]
```

Downloads every game's files from its upstream release (each checked against the manifest's sha256),
lays them out exactly as they land on a stick, and writes `<out>/samples-<date>.tar.gz` (the pack) plus
`<out>/samples-<date>.json` (what is in it - the site's page and `install.sh`'s summary read this). No
third-party libraries needed, only the standard library.

## Releases

`samples-<version>.tar.gz` (+ `.json`) on this repository's [releases page](../../releases) - unpack it
wherever a consuming project stages the sample pack. `nightly` is the rolling build of `develop`.

## Format

The tree inside the pack, and the two Game.ini/pcsx.cfg conventions it relies on, are documented at the
top of `tools/build_samples.py`.

## Licence

GPL-3.0-or-later (`LICENSE`) for the tooling. Every sample game keeps its own licence, named (with a link
to its source) in the pack's own `SAMPLES.md` and in `tools/samples/samples.json`.
