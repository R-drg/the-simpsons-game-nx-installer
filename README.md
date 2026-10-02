# the-simpsons-game-nx installer

The installer page of [the-simpsons-game-nx](https://github.com/R-drg/the-simpsons-game-nx), the Nintendo Switch
port of The Simpsons Game: **https://r-drg.github.io/the-simpsons-game-nx-installer/**

You give it your own copy of the Xbox 360 game (a disc image, or the extracted disc with its `default.xex`) and it
gives you a zip ready to extract into `sdmc:/switch/`. Everything runs in your browser: the game files never leave
your computer.

It is [project8-sw-installer](https://github.com/R-drg/project8-sw-installer) for this game, itself adapted from
[nfsmw-nx-installer](https://github.com/StevensND/nfsmw-nx-installer) by StevensND: the same three steps, the same
disc image reader and zip writer, and the same way of streaming a package of several GB to disk.

## What it does

1. Finds `default.xex` (in the disc image, or in the chosen folder, even one level down) and identifies it by its
   SHA-256 in `release/manifest.json`. the-simpsons-game-nx is compiled from one exact executable (the USA retail
   disc), so the page refuses any other: it tells the same game in another release (by the title ID `45410809` in
   the executable's header) apart from a different game.
2. Downloads `the_simpsons_game.nro` and checks its SHA-256.
3. Writes the zip: the program, `simpsons.toml`, `LICENSES.txt` and, for a first installation, the disc's files
   under `game/` (without the `$SYSTEMUPDATE` partition), all in `the_simpsons_game/`. The update package leaves the
   disc out and needs only `default.xex`.
4. Streams the zip to disk through the service worker, so it never has to fit in memory.

Nothing is made from the disc here: the port translates the game's shaders on the console. So the page needs no
WebAssembly tools.

The page and the downloaded program are cached by the service worker: after the first visit it also works without a
connection.

## Layout

| Path | Contents |
|---|---|
| `index.html`, `app.js` | The page and its interface (English, Spanish, Brazilian Portuguese: `lib/i18n.js`) |
| `worker.js` | Builds the package off the main thread |
| `sw.js` | Offline cache and the download stream |
| `lib/` | Disc image reader, identification and packaging, zip writer |
| `release/` | The list of builds, the settings file and the licences that go into the zip |
| `test/` | A Node script that runs the same code as the page on a synthetic game |

## Deployment

The NRO is not stored here. It is the `the_simpsons_game.nro` asset of a release of
[R-drg/the-simpsons-game-nx](https://github.com/R-drg/the-simpsons-game-nx/releases). The workflow in
`.github/workflows/deploy.yml` tests the package code, downloads the NRO from the latest release, checks its
SHA-256 against `release/manifest.json`, and publishes the page on GitHub Pages. It runs on every push to `main`;
after publishing a new release, run it by hand from the Actions tab. Pages must be set to deploy from GitHub Actions
(Settings > Pages > Source).

To publish a new build: update `nro_sha256` in `release/manifest.json`, copy `release/simpsons.toml` from the
the-simpsons-game-nx release, and rebuild `release/LICENSES.txt` from its `LICENSE` and `NOTICE` (the introduction
at the top of the file stays).

## Testing locally

```sh
node test/package_node.mjs [path/to/default.xex]
cp /path/to/the_simpsons_game.nro release/
python3 serve.py           # http://127.0.0.1:8194/
```

With a path to your own `default.xex`, the test also checks that the manifest knows it.

## Licence

GPL-3.0 (`LICENSE`), as nfsmw-nx-installer. Third-party parts are listed in [NOTICES.md](NOTICES.md).
