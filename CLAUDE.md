# panosplit — panorama split web tool

Small single-page browser tool that splits a panorama into tiles for printing/hanging, with a
tape-measure/DPI layout and a WebGL 3D preview. `index.html` (canvas + WebGL), offline.
Public repo: github.com/bastian74/panosplit.

## Build & test

No build step. Verify logic in node where practical (split/DPI/export math extracts cleanly);
if you must use the in-app browser pane, keep it brief (shared surface). The split/DPI/export
layout math is verified correct — don't regress it.

## Architecture

Single `index.html`: file load (drag-and-drop) → layout/DPI model → canvas tile export (ZIP of
per-tile PNGs) → optional WebGL 3D preview. A WebGL init failure must degrade to a working 2D
editor + export (not crash the app) — that fallback exists as of 2026-07-19, and covers
three.min.js itself failing to load (the only top-level THREE reference is typeof-guarded;
keep it that way).

## Gotchas (see AUDIT-2026-07-19.md)

- Dropping an image OUTSIDE the dropzone must not navigate the page away (it used to, destroying
  layout work) — the whole-page drop handler now loads the image instead.
- Export can exceed browser canvas limits on huge panos — `toBlob` null (Chrome) and thrown
  errors (Firefox/Safari) are both counted/reported, not silently hung. Crops are encoded
  sequentially — one canvas alive at a time — to cap peak memory; keep it that way. Because
  encoding now spans async steps, the source image is pinned at click time (`var img=panoImg`
  in downloadFrames) so a panorama dropped mid-export can't corrupt the remaining crops; draw
  from that capture, never the live global. Transparent PNG input is flattened onto white
  before the JPEG encode.
- Deferred: wrap-mode padding is asymmetric near image edges; no ICC/color-profile handling; a
  display-proxy bitmap would smooth editing of 200 MP+ panos; a printable "hang sheet" from the
  existing tape-measure data is the top feature idea.
- Public repo — docs quality matters for strangers cloning it. Most recent audit: **AUDIT-2026-07-19.md**.
