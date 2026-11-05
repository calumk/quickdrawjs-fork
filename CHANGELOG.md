# Changelog

All notable changes in this fork vs upstream
[quickdrawjs/quickdraw](https://github.com/quickdrawjs/quickdraw) `main`.
Each entry links its upstream issue and PR; unmerged entries are live here
until they land upstream.

## [Unreleased]

### Added — paint order: images are the bottom layer

Layered render order: images, then highlighters, then everything else
(z-index still orders within each layer). Previously images interleaved with
ink by z-index, so photos could cover strokes and highlighter could never sit
over a photo.
Upstream issue: https://github.com/quickdrawjs/quickdraw/issues/30 —
PR: https://github.com/quickdrawjs/quickdraw/pull/31

### Added — pluggable font stacks

`setFontStacks()` / `resetFontStacks()` / `fontStackOf()` plus a `FONT_IDS`
roster: remap any font style id (or register new ids) to a full CSS
font-family stack with one call. Measurement, drawing, the editing textarea
and UI previews all resolve through it; unknown ids fall back to `draw`.
No behavior change unless called.
Upstream issue: https://github.com/quickdrawjs/quickdraw/issues/32 —
PR: https://github.com/quickdrawjs/quickdraw/pull/33

### Added — font picker in the styles popover

Exposes the existing per-shape `font` style (text, notes, geo labels) as a
Hand / Sans / Serif / Mono row following the dash/fill pattern, each option
previewed in its own typeface.
Upstream issue: https://github.com/quickdrawjs/quickdraw/issues/34 —
PR: https://github.com/quickdrawjs/quickdraw/pull/35

### Added — core seam for server-side and export embedders

`setShapePlatform()` backend injection, per-render `drawShape` opts
(`Path2D` class, pre-decoded `images` map, `patternCache` map), shared
`sortShapesForRender()` backing `Editor.shapesSorted()`, and new exports
(`textLayout`, `noteLayout`, `boundsUnion`, …) with types. No behavior
change, no new dependencies.
Upstream issue: https://github.com/quickdrawjs/quickdraw/issues/37 —
PR: https://github.com/quickdrawjs/quickdraw/pull/38

### Added — clipboard overhaul

Memory-first copy/paste (works offline and without clipboard permission,
including across boards on one page); Cmd+V no longer intercepted so the
native `paste` event carries Finder files and raw image data via
`items.getAsFile()`; Quickdraw JSON still pastes from text; silent catches
now warn.
Upstream issue: https://github.com/quickdrawjs/quickdraw/issues/39 —
PR: https://github.com/quickdrawjs/quickdraw/pull/40

### Added — opt-in `disablePanScroll` embed scrolling

`new Editor({ disablePanScroll: true })` / `<Quickdraw disablePanScroll>`:
plain wheel passes through to the page while Ctrl/Meta+wheel still zooms the
board. Off by default; pointer tools (incl. hand drag-pan) unaffected.
Upstream issue: https://github.com/quickdrawjs/quickdraw/issues/41 —
PR: https://github.com/quickdrawjs/quickdraw/pull/42

## Deliberately not included

- **Server-side PNG rendering** lives separately at
  [calumk/quickdrawjs-headless](https://github.com/calumk/quickdrawjs-headless)
  (tracked upstream as https://github.com/quickdrawjs/quickdraw/issues/36):
  it needs a native Skia dependency the monorepo should not carry.
- **Bundled Roboto / Roboto Mono** were tried and reverted: they would
  silently restyle every existing board's sans/mono text. System stacks stay
  as upstream; pin faces via `setFontStacks()` instead.
