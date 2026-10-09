# Muon3D Slicer: printer builds

Release builds of the copy of the Muon3D Slicer that a Muon printer serves itself. This repository holds no source:
each [release](https://github.com/Muon-3D/muon3d-slicer-dist/releases) is one build, as a zip.

## What the zip is

`muon3d-slicer-printer-<commit>.zip` is the Muon3D Slicer, the web app that slices 3D models in the browser, built
from one commit of the slicer to be served by the printer:

- it is served by the printer's own web server at `http://<printer>/slicer/`, beside Fluidd, which opens it from its
  **Slice** page; it works with no account and no internet connection;
- it slices with the single-threaded slicing engine (a page served over plain HTTP on a local network cannot use the
  multithreaded one), and it keeps models, projects and results in memory only, for as long as the page is open;
- it has no service worker and no sign-in.

The zip holds:

| Path | What |
|---|---|
| `slicer/` | The site, installed as `/opt/muon-slicer/www/slicer/` |
| `LICENSE` | The Muon3D Slicer's licence |
| `ENGINE-LICENSE`, `ENGINE-NOTICE`, `ENGINE-SOURCE.md` | The slicing engine's licence (AGPL-3.0), its notice, and how to get its source; the same files are in `slicer/engine/<version>/` |
| `SIZES.txt` | What the copy takes on the printer, by part |
| `SHA256SUMS` | The SHA-256 of every other file in the zip |

The same commit always gives the same zip, byte for byte.

## Licences

**The slicing engine** (Muon3D Slicer Engine, based on OrcaSlicer) is a separate program that runs in your browser.
It is free software under the **GNU Affero General Public License v3.0**: you may use, study, share and change it
under that licence. Its source is at <https://github.com/Muon-3D/muon3d-slicer-engine>; each engine release there has
the complete source of that release as assets, and `ENGINE-SOURCE.md` in the zip says exactly what the source is
and how to build it.

**The Muon3D Slicer itself**, the web app around the engine, is proprietary. Its `LICENSE` says:

> Copyright (c) 2026 Muon 3D Technologies Limited. All rights reserved.
>
> This software and its source code are proprietary and confidential. No licence to use, copy, modify or distribute
> them is granted except by written agreement with Muon 3D Technologies Limited.

Its source is not published. The free software inside the app (npm packages and fonts) is listed with each licence
on the app's own licences page, `slicer/licenses.html` (About → Licences in the app).

## Releases

Each release is tagged `printer-<commit>`, the first 12 characters of the slicer commit it was built from, and has
three assets: the zip, its `SIZES.txt`, and a `SHA256SUMS` of both. The zip's SHA-256 is also in the release notes.
Releases are immutable: an asset is never replaced, and a new build is a new release.

To check a download:

```sh
sha256sum -c --ignore-missing SHA256SUMS
unzip muon3d-slicer-printer-<commit>.zip -d copy && (cd copy && sha256sum -c SHA256SUMS)
```

## How MuonOS uses it

MuonOS, the printer's operating system, fetches the zip when its image is built, never on the printer. Its
`muon-slicer` recipe pins one release by two values, changed together:

```toml
slicer_download_url = { default = "https://github.com/Muon-3D/muon3d-slicer-dist/releases/download/printer-<commit>/muon3d-slicer-printer-<commit>.zip" }
slicer_sha256 = { default = "<the zip's sha256, from the release notes or SHA256SUMS>" }
```

The build downloads the zip, refuses it unless its SHA-256 is the pinned one, checks every file against the zip's
own `SHA256SUMS`, and installs `slicer/` as `/opt/muon-slicer/www/slicer/`. Updating the printer's slicer is a new
pin in MuonOS, which reaches printers with the next update of their image.
