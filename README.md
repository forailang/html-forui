# html-forui

The HTML/browser adapter for [forui](https://github.com/forailang/forui).
Takes a forui `ViewNode` tree and turns it into HTML — either a full
page string (server-side rendering) or a stream of diff patches
applied to the live DOM (client-side reactive updates).

If you're building a forui app for the browser or producing
server-rendered HTML, this is the package you depend on alongside
`forui` itself.

## What it ships

- `htmlRender` — render the current app's ViewNode tree to HTML and
  inject into `#app`. The browser entry point.
- `renderToString` — server-side: render a forui app at a given path
  synchronously and return the HTML string. Inject into the page
  shell's `<div id="app">`.
- `initWebRouter` — wire the forui router to the browser History API
  (`pushState` + `popstate`). Call once after `mount()` in your
  client main.
- DOM-patch primitives (`patchPathForOps`, `findNodeAtPath`, …) used
  by the live-update path; rarely called directly from app code.

## Install

```toml
[dependencies]
Forui     = "https://github.com/forailang/forui"
HtmlForui = "https://github.com/forailang/html-forui"
```

forai will shallow-clone both into `~/.fai/cache/git/github.com/forailang/`
on first build and reuse them after that. To refresh, delete the
cache directory.

For local iteration on either library, swap to a relative `file://`
path:

```toml
[dependencies]
Forui     = "file://../forui"
HtmlForui = "file://../html-forui"
```

## Hello, world

The smallest browser app: a single `Label` rendered into `#app`.

```fai
use { mount, ViewNode } from Forui
use { Label } from Forui.view
use { htmlRender, initWebRouter } from HtmlForui

# Top-level component.
def App
    @return ViewNode
do
  Label('Hello, world.')
end

def main
    @return Void
do
  mount(App)
  initWebRouter()
  htmlRender()
end
```

The `forui-fullstack` template wires this up end-to-end with routing,
RPC, and a server entry point — see
[github.com/forailang/forui-fullstack](https://github.com/forailang/forui-fullstack).

## Building and testing

```bash
fai check                # format + type check
fai test                 # 68 inline tests
```

There is no separate build step for html-forui — it compiles inside
whichever app consumes it.

## License

Apache-2.0.
