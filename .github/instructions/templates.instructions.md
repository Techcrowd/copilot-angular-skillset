---
applyTo: "**/*.component.html,**/*.page.html"
description: Angular template rules — new control flow, signals, a11y
---

# Template rules

## Control flow — use ONLY new syntax

```html
@if (user(); as u) {
  <app-profile [user]="u" />
} @else if (loading()) {
  <app-spinner />
} @else {
  <app-empty />
}

@for (item of items(); track item.id) {
  <app-row [item]="item" />
} @empty {
  <p>No items.</p>
}

@switch (status()) {
  @case ('loading') { <app-spinner /> }
  @case ('error')   { <app-error /> }
  @default          { <app-content /> }
}

@let displayName = user()?.name ?? 'Guest';
<h1>Hello {{ displayName }}</h1>
```

Never use `*ngIf`, `*ngFor`, `*ngSwitch`, `[ngSwitch]`. They are deprecated for this project.

## `@for` rules
- `track` is mandatory and must be a stable identifier (prefer `item.id`)
- `track $index` only for static lists that never reorder
- Use `@empty` instead of separate `@if (items().length === 0)`

## `@defer`

```html
@defer (on viewport) {
  <app-heavy-chart [data]="data()" />
} @placeholder (minimum 200ms) {
  <div class="skeleton h-64"></div>
} @loading (after 100ms; minimum 300ms) {
  <app-spinner />
} @error {
  <app-load-error />
}
```

Use `@defer` for: charts, rich text editors, maps, rarely-shown panels, anything below the fold.

## Signal access in templates
- Call signals as functions: `{{ user().name }}`, `[disabled]="loading()"`
- Cache repeated reads with `@let`: `@let u = user(); ... {{ u?.name }}`
- Don't call signals in tight loops without caching

## Bindings
- Prefer property binding `[disabled]` over attribute binding `attr.disabled`
- Use `class.x` and `style.y` bindings for conditional styling
- Boolean attrs: `[attr.aria-expanded]="opened()"` for ARIA, `[disabled]="disabled()"` for DOM

## Accessibility — required
- Buttons must have visible text OR `aria-label`
- Form inputs must have an associated `<label for="...">`
- `<img>` requires `alt` (empty `alt=""` for decorative)
- Focus order must follow visual order — no `tabindex` > 0
- Don't use `<div>` / `<span>` for clickable elements — use `<button>` or `<a>`

## What to avoid
- `[innerHTML]` with user-provided strings (XSS) — sanitize via `DomSanitizer` only when truly needed
- Long inline expressions — extract into `computed()` in the component
- Logic in templates beyond simple comparisons
- `ng-container` chains where `@if` would do
