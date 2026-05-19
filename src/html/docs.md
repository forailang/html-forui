# HtmlForui.html

`HtmlForui.html` contains the low-level HTML renderer used by the public
`HtmlForui` adapter entry points. Most applications should import from
`HtmlForui`, not this submodule.

Use `renderToHtml(node)` when you already have a `ViewNode` tree and need a
plain HTML string:

```fai
use { renderToHtml } from HtmlForui.html

let html = renderToHtml(App())
```

Use `HtmlForui.renderToString(App, path)` instead when rendering a routed app
from a server request, because it sets the route path and uses Forui's SSR hook
isolation.

`renderPatchHtml(node, path)` renders HTML with `data-fai-path` attributes used
by the browser patch adapter. The rest of this module is mostly implementation
detail:

- component renderers: `renderButton`, `renderTextInput`, `renderLink`,
  `renderRouter`, `renderToggle`, and related functions
- attribute/style helpers: `nodeAttrs`, `pathAttr`, `keyAttr`, `classAttr`,
  `propsToStyle`, and `propOr`
- tree/path helpers: `renderChildrenInternal`, `childPath`, and test helpers

Extend these functions when adding support for a new Forui `ViewNode.kind`.
Ordinary app code should use Forui components and modifiers instead.
