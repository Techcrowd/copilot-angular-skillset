---
mode: agent
description: Migrate templates from *ngIf / *ngFor / *ngSwitch to new @if / @for / @switch
---

Migrate the selected `.html` template (or all templates in the selected folder)
from structural directives to **new control flow** syntax.

## Transformations

### `*ngIf`

```html
<!-- BEFORE -->
<div *ngIf="user">{{ user.name }}</div>
<div *ngIf="loading; else content">Loading...</div>
<ng-template #content><div>Done</div></ng-template>

<!-- AFTER -->
@if (user) {
  <div>{{ user.name }}</div>
}

@if (loading) {
  <div>Loading...</div>
} @else {
  <div>Done</div>
}
```

### `*ngIf` with `as`

```html
<!-- BEFORE -->
<div *ngIf="user$ | async as user">{{ user.name }}</div>

<!-- AFTER -->
@if (user(); as user) {
  <div>{{ user.name }}</div>
}
```
(Assumes signal-based source; if Observable, convert source with `toSignal()` first.)

### `*ngFor`

```html
<!-- BEFORE -->
<li *ngFor="let item of items; trackBy: trackById; let i = index">
  {{ i }}: {{ item.name }}
</li>

<!-- AFTER -->
@for (item of items; track item.id; let i = $index) {
  <li>{{ i }}: {{ item.name }}</li>
} @empty {
  <li>No items.</li>
}
```

### `*ngSwitch`

```html
<!-- BEFORE -->
<div [ngSwitch]="status">
  <div *ngSwitchCase="'a'">A</div>
  <div *ngSwitchCase="'b'">B</div>
  <div *ngSwitchDefault>?</div>
</div>

<!-- AFTER -->
@switch (status) {
  @case ('a') { <div>A</div> }
  @case ('b') { <div>B</div> }
  @default    { <div>?</div> }
}
```

### `ng-container` cleanup

After migration, `<ng-container>` that only existed to host `*ngIf` / `*ngFor` becomes unnecessary:

```html
<!-- BEFORE -->
<ng-container *ngIf="user">
  <h1>{{ user.name }}</h1>
  <p>{{ user.email }}</p>
</ng-container>

<!-- AFTER -->
@if (user) {
  <h1>{{ user.name }}</h1>
  <p>{{ user.email }}</p>
}
```

## Rules

- `track` is **mandatory** in `@for`. Prefer stable IDs; fall back to `$index` only for static lists.
- Convert `trackBy: trackFn` calls into inline expressions where possible: `track item.id`
- `let i = index` becomes `let i = $index`. Other contextual vars: `$first`, `$last`, `$even`, `$odd`, `$count`
- Don't change indentation/styling of unrelated HTML
- After migrating a template, remove unused imports from the component (`NgIf`, `NgFor`, `NgSwitch`, `AsyncPipe` if no longer used)
- Run `ng generate @angular/core:control-flow` first IF you want Angular's official migration to do the bulk — then this prompt cleans up edge cases

## Process

1. List every `*ngIf` / `*ngFor` / `*ngSwitch` / `[ngSwitch]` occurrence in chat
2. Convert one template at a time
3. Update the component's `imports: []` array to remove unused directives
4. Verify no broken templates by re-reading the result
