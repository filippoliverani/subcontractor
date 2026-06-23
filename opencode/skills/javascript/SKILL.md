---
name: javascript
description: >-
  Write, review, and refactor JavaScript in a Rails app using Stimulus and
  Hotwire (Turbo). Use when adding interactivity to ERB/Turbo views, creating or
  editing Stimulus controllers, wiring data-controller/data-action/data-target
  attributes, debugging "controller not connecting" issues, handling Turbo
  Frame/Stream lifecycle in JS, or deciding between a Stimulus controller and
  plain JS. Triggers: "stimulus controller", "data-action", "data-target",
  "hotwire", "turbo frame", "importmap/jsbundling", "add JS behavior to this
  view".
---

# JavaScript

Write idiomatic Stimulus for a Hotwire-first Rails app. Default to HTML-over-the-wire (Turbo) and reach for JS only to enhance behavior, never to render state the server can render.

## Core principle

Stimulus is for **sprinkling behavior onto server-rendered HTML**, not building a client-side app. The DOM is the source of truth. If you're keeping significant state in JS, reconsider — push it to the server with Turbo, or use a `<template>` / data attributes.

## Naming & file conventions

- One controller per file, identifier in kebab-case (`dropdown_menu_controller.js` → `data-controller="dropdown-menu"`).
- Class name in PascalCase ending in `Controller`: `export default class extends Controller`.
- Keep controllers small and single-purpose

## Controller anatomy — use the official API, not ad-hoc JS

Always prefer Stimulus primitives over manual DOM queries:

```js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = ["input", "output"]
  static values  = { url: String, open: { type: Boolean, default: false } }
  static classes = ["active"]

  connect()    { /* setup when element enters DOM */ }
  disconnect() { /* teardown — remove listeners, timers, observers */ }

  submit(event) {
    event.preventDefault()
    this.outputTarget.textContent = this.inputTarget.value
  }

  openValueChanged(open) {            // value change callback
    this.element.classList.toggle(this.activeClass, open)
  }
}
```

Rules:
- Use `static targets` + `this.xTarget` / `this.xTargets` / `this.hasXTarget` instead of `querySelector`.
- Use `static values` for configuration/state passed from the server (`data-<id>-<name>-value`). Use `*ValueChanged` callbacks to react to changes — this is the idiomatic way to keep DOM in sync.
- Use `static classes` instead of hardcoding CSS class strings, so styling stays in the view.
- Bind events in the HTML with `data-action`, not `addEventListener`, whenever possible: `data-action="click->dropdown-menu#toggle"`.

## Lifecycle & cleanup (the #1 source of bugs with Turbo)

Because Turbo swaps DOM without full page reloads, controllers connect/disconnect repeatedly. Always pair setup with teardown:

- Anything added in `connect()` (event listeners on `window`/`document`, `setInterval`, `IntersectionObserver`, third-party plugin init) **must** be removed in `disconnect()`.
- Store handler references so they can be removed: `this.onResize = this.onResize.bind(this)` in the constructor or use arrow-function fields.
- Never assume a controller runs once — it can connect many times in one page session.
- Prefer `data-action` with global targets (`resize@window->`, `keydown@document->`) over manual `window.addEventListener`; Stimulus handles cleanup for you.

## Turbo integration

- Don't fight Turbo navigation with manual JS routing. Let Turbo Drive handle links/forms.
- For partial updates use **Turbo Frames** (`<turbo-frame>`) and **Turbo Streams** before writing JS.
- React to Turbo lifecycle via events when needed: `turbo:load`, `turbo:frame-load`, `turbo:before-stream-render`, `turbo:submit-end`. Wire them through `data-action` on a controller, e.g. `data-action="turbo:submit-end->form#reset"`.
- For one-off scripts that must run on every visit, listen for `turbo:load`, not `DOMContentLoaded` (which only fires on full loads).

## Passing data from Rails to Stimulus

- Use value attributes generated from the server, not inline `<script>` blobs:
  ```erb
  <div data-controller="poll"
       data-poll-url-value="<%= status_path %>"
       data-poll-interval-value="5000"></div>
  ```
- For larger payloads, use a JSON `data-*-value` (Object/Array types) or a `<template>` element — never `gon`-style globals or `window.*` assignments.
- Escape correctly: rely on Rails' HTML escaping; never interpolate user data into JS strings.

## Things to avoid

- jQuery, or `$`-style DOM manipulation. Use native DOM + Stimulus targets.
- Global state on `window`. Pass data via attributes; share between controllers via custom events (`this.dispatch("name", { detail })`) or the Outlets API.
- `setTimeout` hacks to "wait for the DOM" — use `connect()`, `targetConnected()`, or Turbo events.
- Business logic in controllers. Keep them thin; move heavy logic to plain JS modules in `app/javascript/lib/` and import them.
- Querying outside `this.element` unless intentional. Controllers should be scoped to their element.

## Cross-controller communication

- **Events** for loose coupling: `this.dispatch("selected", { detail: { id } })` → listen with `data-action="other-controller:selected->me#handle"`.
- **Outlets** when one controller needs to call another's methods directly (`static outlets = ["modal"]`).
- Avoid reaching into `this.application.getControllerForElementAndIdentifier(...)` unless there's no cleaner option.

## Testing & debugging checklist

When a controller "doesn't work":
1. Confirm the identifier matches the filename (kebab-case vs snake_case mismatch is the usual culprit).
2. Confirm it's registered (importmap pinned / bundled and present in `controllers/index.js`).
3. Check `data-action` syntax: `event->identifier#method`.
4. Verify it's not a Turbo lifecycle issue — does it work on full reload but not after a Turbo visit? → missing `turbo:load` handling or teardown.
5. Use `this.application.debug = true` (or set `data-turbo="false"` temporarily) to isolate Turbo vs Stimulus.

## Style

- Match the project's existing module system and lint config. Don't introduce new tooling.
- Prefer modern syntax: `const`/`let`, arrow functions, optional chaining, class fields.
- Keep ERB and JS concerns separated: behavior in the controller, markup/classes in the view, config in `data-*-value`.

