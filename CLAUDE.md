# html-forui — HTML/browser renderer for forui

html-forui is the browser-side adapter for the
[forui](https://github.com/forailang/forui) UI framework. It turns
forui's `ViewNode` trees into HTML — either a full string for
server-side rendering or a stream of diff patches applied to the live
DOM.

This package depends on `forui` and is meant to be paired with it in
any browser-targeting app.

## Language Reference

See [forai language.md](https://github.com/forailang/forai/blob/main/language.md)
for full forai syntax and features.

## Public API

```fai
use { htmlRender, renderToString, initWebRouter } from HtmlForui

# Browser: build the current ViewNode tree, walk the diff vs the
# previous render, and apply patches to #app.
htmlRender()

# Server: render the app at a path synchronously (signal loaders
# execute during render) and return a plain HTML string.
let body = renderToString(App, '/posts/42')

# Wire the forui router to the browser History API. Call once after
# the first mount() / htmlRender().
initWebRouter()
```

## File Structure

```
src/
  htmlforui.fai     — entry-point module: htmlRender, renderToString,
                       initWebRouter, dom-patch helpers (patchPathForOps,
                       findNodeAtPath, childPathFor, parentPath).
  html/
    html.fai        — node renderers: one per ViewNode kind
                       (renderLabel, renderButton, renderImage, …),
                       attribute building, child traversal, diff-op
                       application (renderPatchHtml).
```

A directory under `src/` is one forai module. Consumers import the
top-level helpers from `HtmlForui`; the `html` submodule is for
internal use and reused by the entry-point file.

## Key Design Notes

- **Two render paths.** `renderToString` builds a complete HTML
  string up-front (used for SSR + the initial page payload).
  `htmlRender` runs in the browser: it diffs the new ViewNode tree
  against the last one and applies a list of `DiffOp` patches via
  `renderPatchHtml`, mutating only what changed.

- **Path addressing.** Each ViewNode is addressed by a dot-path
  (`'1.2.3'`) describing its position in the tree. Patches are
  keyed by these paths so the browser-side applier can locate the
  target DOM node without re-walking the whole tree.

- **Insert/remove ops act on the parent.** When `DiffOp::Insert` or
  `Remove` arrives, the path actually identifies the parent
  container — `patchPathForOps` accounts for that and returns the
  correct apply-target.

- **Web router integration via host functions.** `initWebRouter`
  expects three platform-provided JS shims: `getLocationPath()`,
  `pushHistoryState(path)`, and `historyBack()`. The hosting page
  also needs a `popstate` listener that calls
  `setPathFromPlatform(window.location.pathname)`. The `forui-fullstack`
  template ships these in its runtime shell.

- **Single dependency: forui.** This package re-exports nothing from
  forui — consumers import view types and signals directly from
  `Forui` / `Forui.view` / `Forui.router` / etc.

## Testing

Tests are inline in the same files as their functions (forai
convention). Run with `fai test` — the suite covers every
`render*` function, the diff-op patch path, and attribute building.

68 tests, 100% coverage at the time of v0.1.0.

## Roadmap

- **Streaming SSR.** Currently `renderToString` is synchronous and
  blocks on signal loaders. A streaming variant that flushes a
  shell + filled regions as data arrives would shorten time-to-first-byte
  for slow loaders.
- **Custom element support.** Today every `ViewNode.kind` maps to a
  built-in renderer. A registration hook so apps can add their own
  kinds (and ship their own renderer pair) is the natural extension.
