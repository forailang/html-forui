# HtmlForui

`HtmlForui` is the HTML/browser adapter for Forui. It turns a Forui `ViewNode`
tree into HTML for server-side rendering and patches the live browser DOM after
client-side state changes.

Most browser apps import the adapter entry points directly:

```fai
use { mount } from Forui
use { htmlRender, initWebRouter } from HtmlForui
use { App } from platforms.web

def main
    @return Void
do
    initWebRouter()
    mount(App, htmlRender)
end
```

`initWebRouter()` wires `Forui.router` to the browser History API. Call it once
in the web client entry before the first mount so the router starts from the
current URL path. `mount(App, htmlRender)` renders the app and registers
`htmlRender` as the adapter that receives future diff patches.

## Server Rendering

Use `renderToString(App, path)` from the server target to render the app for a
request path. It returns the HTML fragment that belongs inside the page shell's
`<div id="app">`.

```fai
use std.http.server

use { renderToString } from HtmlForui
use { App, makeHtmlShell } from platforms.web

server.get(r, '*') do with req HttpRequest
    server.html(200, makeHtmlShell(renderToString(App, req.path)))
end
```

`renderToString` delegates to `Forui.renderSSR`. Loader-backed signals render in
their loading state during SSR and then load on the browser client.

Polling through `Forui.signal.usePoll` is owned by Forui and the language async
timer runtime. `HtmlForui` does not expose a timer API; poll handlers mutate
signals or call `reload()`, and `htmlRender` receives the resulting diff patches
through the normal render cycle.

## Shell Assets

The HTML shell should include:

```html
<link rel="stylesheet" href="/forui.css">
<div id="app">{{APP_HTML}}</div>
<script src="/fai-runtime.js"></script>
```

`forui.css` provides the base styles for framework classes such as
`fai-vstack`, `fai-hstack`, `fai-button`, and `fai-label`. Inline styles from
Forui modifiers still render on each node, so app-specific modifiers override
the defaults.

`fai-runtime.js` is the generated browser runtime used by the compiled client
artifact. It hosts the WASM module, dispatches events, applies DOM patches, and
bridges router history functions used by `initWebRouter`.

## Direct Rendering

Most app code should use `htmlRender` for browser mounting and
`renderToString` for server rendering. Use `HtmlForui.html.renderToHtml`
directly only when you already have a `ViewNode` tree and need a plain HTML
string outside the normal app lifecycle, such as tests, static generation, or a
custom adapter.

`HtmlForui.html.renderPatchHtml` is patch-aware and adds stable
`data-fai-path` attributes for DOM updates. It is mainly for adapter internals.

## Internal Renderer Helpers

The `HtmlForui.html` namespace exposes the renderer implementation. Functions
such as `renderNodeHtml`, `renderChildrenInternal`, `nodeAttrs`, `propsToStyle`,
`renderButton`, `renderTextInput`, and `renderLink` are useful when extending or
testing the adapter, but ordinary applications should not call them directly.
