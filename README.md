# OSIRIS.EXE

Transmedia archive site by **Frank Sanchez — DISK / Disk Darián** (DSB Labs). Chicago · Humboldt Park · Boricua roots.

**Live:** https://kingpiragua.github.io/OSIRIS/

## Pages

| Page | Description |
|---|---|
| [`index.html`](index.html) | Fullscreen WebGL2 raymarched fly-through. Spacebar toggles play/pause. Small `ARCHIVE ▸` link, top-left. |
| [`archive.html`](archive.html) | "Recovered archive" gallery — 7 artworks as terminal file cards over the live shader, CRT scanlines, click any card for full size. |

Both are single self-contained HTML files. No build step, no dependencies, no bundler — open them directly or serve the folder.

## Structure

```
index.html            shader page (inline CSS + GLSL + JS)
archive.html          gallery page (inline CSS + GLSL + JS)
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
