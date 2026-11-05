# @quickdrawjs/core (fork)

> Community fork of [quickdrawjs/quickdraw](https://github.com/quickdrawjs/quickdraw)'s
> core engine, tracking upstream `main` plus the open feature PRs (image-bottom
> stacking, pluggable font stacks, font picker, clipboard overhaul, embed
> scrolling, server-renderer seam). Installable directly for local use while
> those land upstream:
>
> ```bash
> npm install github:calumk/quickdrawjs-fork
> ```

The framework-free heart of [Quickdraw](https://tryquickdraw.com) — an MIT-licensed
infinite-canvas whiteboard engine. Zero dependencies, zero build step: plain ESM
that runs in any modern browser.

If you're using React or React Native, you probably want
[`@quickdrawjs/react`](https://www.npmjs.com/package/@quickdrawjs/react) or
[`@quickdrawjs/react-native`](https://www.npmjs.com/package/@quickdrawjs/react-native)
instead — they wrap this engine.

## Install

```bash
npm install @quickdrawjs/core
```

## Quick start

```js
import { createQuickdraw } from '@quickdrawjs/core'
import '@quickdrawjs/core/quickdraw.css'

const board = createQuickdraw({
  container: document.getElementById('board'),
  theme: 'light', // or 'dark'
  grid: 'lines',  // 'none' | 'lines' (default) | 'ruled' | 'dots' | 'crosses' | 'iso'
})

// the document store emits a diff after every change
board.editor.store.listen((diff, source) => {
  console.log('changed', diff)
})

// later
board.destroy()
```

That's a complete whiteboard: pen with pressure/velocity ink, highlighter,
shapes, arrows with bendable curves, text, sticky notes, images
(paste/drop/pick), eraser, laser pointer, selection with resize/rotate,
pan/zoom/pinch, undo/redo, PNG export, a responsive floating toolbar, light and
dark themes, grid backdrops, and full keyboard shortcuts.

## Theme, grid, and the board menu

Both live on the editor and in the ⋮ board menu, so users can switch them
without you building any chrome:

```js
board.editor.setTheme('dark')     // emits 'theme'
board.editor.setGrid('dots')      // 'none' | 'lines' (default) | 'ruled' | 'dots' | 'crosses' | 'iso', emits 'grid'
board.editor.clearBoard()         // one undoable step (⇧⌘⌫)

// mirror in-board switches into your own app state
board.editor.on('theme', () => setMyTheme(board.editor.theme.id))
board.editor.on('grid', () => setMyGrid(board.editor.grid))
```

If your app owns its own theme chrome, drop the in-board switches:

```js
createQuickdraw({ container, themeToggle: false, gridControl: false })
```

A small "Quickdraw" mark sits in the board's corner. Keeping it helps people
find the project; `watermark: false` removes it — no purchase required.

The grid spacing adapts to the zoom (doubling and halving around a 40px page
step, majors every fifth) and travels into PNG exports that keep the paper.

## Headless / custom UI

`createQuickdraw` is just `new Editor(...)` plus the stock toolbar. Use the
editor bare and bring your own chrome:

```js
import { Editor } from '@quickdrawjs/core'
import '@quickdrawjs/core/quickdraw.css'

const editor = new Editor({ container, theme: 'dark', grid: 'dots' })
editor.setTool('draw')
editor.setStyle('color', 'blue')
editor.on('selection', () => console.log([...editor.selection]))
```

## The document

The store is a flat map of immutable records. Every mutation happens in a
transaction and emits a diff — `{ added, removed, updated: {id: [from, to]} }` —
which is also the wire format for sync and the shape of undo history:

```js
const snapshot = editor.store.getSnapshot()   // serialize (JSON-safe)
editor.store.loadSnapshot(snapshot)           // restore

// real-time sync: ship diffs both ways
editor.store.listen((diff) => socket.send(JSON.stringify(diff)), { source: 'user' })
socket.onmessage = (e) => editor.store.applyDiff(JSON.parse(e.data), 'remote')
```

Remote diffs don't enter local undo history, so collaborative undo stays sane.

## Export

```js
const blob = await editor.exportImage({ background: true, scale: 2 })
```

See the [repository README](https://github.com/quickdrawjs/quickdraw) for the
full API, data model and guides.

## Configuration reference (fork additions)

Everything here is opt-in; defaults match upstream exactly. See
[CHANGELOG.md](./CHANGELOG.md) for rationale and upstream tracking.

```js
import {
  Editor, FONT_IDS, setFontStacks, resetFontStacks, fontStackOf,
} from '@quickdrawjs/core'

// --- editor construction ---
const editor = new Editor({
  container,                 // required HTMLElement
  store,                     // optional Store (default: new Store())
  theme: 'light',            // 'light' | 'dark'
  grid: 'lines',             // 'none' | 'lines' | 'ruled' | 'dots' | 'crosses' | 'iso'
  readonly: false,           // lock all input (also freezes wheel)
  camera,                    // optional { x, y, z } initial viewport
  styles,                    // optional pen defaults (see style keys below)
  geoKind: 'rectangle',      // default geo shape
  disablePanScroll: false,   // true: plain wheel passes to the page,
                             // Ctrl/Meta+wheel still zooms the board
})

// --- fonts ---
FONT_IDS // ['draw', 'sans', 'serif', 'mono']
setFontStacks({
  sans: "'Inter', system-ui, sans-serif",   // remap an id (or add new ids)
  mono: "'JetBrains Mono', ui-monospace, monospace",
})
fontStackOf('sans') // resolved CSS font-family string (unknown ids -> draw)
resetFontStacks()   // restore bundled defaults
// process-global: call once at startup. In the browser, load matching faces
// with @font-face so measurement matches drawing.

// --- per-shape styles (editor.setStyle(key, value) or props) ---
// color: 'black' | 'grey' | 'light-violet' | 'violet' | 'blue' |
//        'light-blue' | 'yellow' | 'orange' | 'green' | 'light-green' |
//        'light-red' | 'red'            (text, notes, geo labels use font:)
// font:  'draw' (hand, bundled) | 'sans' | 'serif' | 'mono'  (text/note/geo)
// size:  's' | 'm' | 'l' | 'xl'
// dash:  'draw' (hand) | 'solid' | 'dashed' | 'dotted'
// fill:  'none' | 'semi' | 'solid' | 'pattern'      (geo only)
// opacity: 0..1
```

## License

MIT
