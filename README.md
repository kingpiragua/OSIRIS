# OSIRIS.EXE

Transmedia archive site by **Frank Sanchez — DISK / Disk Darián** (DSB Labs). Chicago · Humboldt Park · Boricua roots.

**Live:** https://kingpiragua.github.io/OSIRIS/

## Pages

| Page | Description |
|---|---|
| [`index.html`](index.html) | Fullscreen WebGL2 raymarched fly-through. Spacebar toggles play/pause. Small `ARCHIVE ▸` link, top-left. |
| [`archive.html`](archive.html) | "Recovered archive" gallery — 7 artworks as terminal file cards over the live shader, CRT scanlines, click any card for full size. |
| [`clock.html`](clock.html) | Recovered-signal clock with gravitational lensing, Signal Bleed colors, and exact local time. |
| [Archive Node 01](https://kingpiragua.github.io/osiris-exe/) | External handoff into the canonical OSIRIS.EXE narrative archive. |

## Canonical experience path

1. **Portal** — enter through the WebGL fly-through on this repository’s live page.
2. **Recovered Gallery / Clock** — explore the atmospheric archive nodes in any order.
3. **Archive Node 01** — follow `[ ENTER ARCHIVE NODE 01 ]` into the canonical boot rite and fragment sequence.
4. **Archive Engine v2** — optional Signal/Network lore node inside the terminal.
5. **Recovered Signal** — conclude with the Pale Horse Protocol motion comic.

This repository is the atmospheric front door. [`kingpiragua/osiris-exe`](https://github.com/kingpiragua/osiris-exe) is the canonical narrative archive.

All local pages are single self-contained HTML files. No build step, no dependencies, no bundler — open them directly or serve the folder.

## Structure

```
index.html            portal + links to gallery, clock, and Archive Node 01
archive.html          recovered gallery (inline CSS + GLSL + JS)
clock.html            recovered-signal clock (inline CSS + canvas + JS)
assets/
  file_001.png …      artwork, file_001 through file_007
LICENSE               CC0 1.0 — repo contents
LICENSE-shader.txt    MIT — shader and renderer only
```

## Running locally

Open `index.html` in any browser. Requires WebGL2; the archive page falls back to a plain black background without it.

To serve over HTTP instead:

```bash
python3 -m http.server 8000
# then visit http://localhost:8000
```

## Deployment

GitHub Pages, deploying from `main` at `/ (root)`. Pushing to `main` republishes the site.

## Credits and licensing

The GLSL shader and the `Renderer` / `PointerHandler` / `Editor` classes come from the pen **"Yotta" by Matthias Hurrle ([@atzedent](https://codepen.io/atzedent))** — https://codepen.io/atzedent/pen/bNBRGbR — used under the MIT License. Attribution comments are preserved in the source; the full license text is in [`LICENSE-shader.txt`](LICENSE-shader.txt).

Artwork in `assets/` is original work by Frank Sanchez.

Everything else in this repository is released under [CC0 1.0](LICENSE).
