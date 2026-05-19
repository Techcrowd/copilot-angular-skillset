# GitHub Copilot — Project Instructions

This is a large Angular application on **v20**, actively migrating to **v21**.
All code suggestions MUST follow modern Angular patterns described below.

---

## Stack

- **Angular 20** (target: 21) — standalone APIs everywhere
- **TypeScript 5.6+** with `strict: true`, `noUncheckedIndexedAccess: true`
- **RxJS 7** — used sparingly; **signals are the default reactive primitive**
- **State management**: [signals / NgRx Signal Store / NGXS — pick one and update this line]
- **Styling**: [Tailwind v4 / SCSS / Angular Material — pick one]
- **Testing**: [Jest / Karma + Jasmine — pick one]
- **Build**: `@angular/build` (esbuild + vite), `application` builder
- **Forms**: Reactive forms (no Template-driven); experimenting with Signal Forms in v21

---

## Core principles (apply to EVERY suggestion)

1. **Standalone everything.** Never create `NgModule`. Components, directives,
   pipes are standalone. Bootstrap with `bootstrapApplication()`.
2. **Signals first.** Use `signal()`, `computed()`, `linkedSignal()`, `resource()`,
   `httpResource()`, `rxResource()`. Use `effect()` sparingly.
3. **`inject()` for DI.** Never use constructor parameter injection.
4. **New control flow** in templates: `@if`, `@for` (with `track`), `@switch`, `@let`.
   Never use `*ngIf`, `*ngFor`, `*ngSwitch`, `ngTemplateOutlet` when `@if/@for` work.
5. **Signal-based component API**: `input()`, `input.required()`, `output()`,
   `model()`, `viewChild()`, `viewChildren()`, `contentChild()`, `contentChildren()`.
   Never use `@Input()`, `@Output()`, `@ViewChild()` decorators.
6. **`OnPush` change detection** by default on every component.
7. **Lazy load by route**: `loadComponent` / `loadChildren` with dynamic `import()`.
8. **`@defer`** for heavy below-the-fold components.
9. **Zoneless-ready code**: never rely on Zone.js triggering CD. Use signals or
   explicit `markForCheck()`.

---

## File structure & naming

Feature-based folders. Each feature is self-contained:

```
src/app/
├── core/                       # singletons, app-wide services, interceptors
├── shared/                     # reusable standalone components, directives, pipes
├── features/
│   └── <feature>/
│       ├── <feature>.routes.ts
│       ├── <feature>.store.ts            # signal store (if needed)
│       ├── pages/
│       │   └── <name>.page.ts/html/scss  # routed components
│       ├── components/
│       │   └── <name>.component.ts/html/scss
│       ├── services/
│       │   └── <name>.service.ts
│       └── models/
│           └── <name>.model.ts
└── app.config.ts               # ApplicationConfig with providers
```

**Naming rules:**
- Files: `kebab-case.{role}.ts` — e.g. `user-profile.component.ts`
- Classes: `PascalCase` matching filename — `UserProfileComponent`
- Selectors: `app-<kebab-case>` for components, `app<PascalCase>` for directives
- Tests: co-located `*.spec.ts` next to source
- Routes: `<feature>.routes.ts` exporting `default` = `Routes`
- One public class per file

---

## Hard anti-patterns — NEVER suggest these

| Don't | Do instead |
|---|---|
| `NgModule` | Standalone component/directive/pipe + `providers` in routes or `app.config.ts` |
| `@Input() foo: string` | `foo = input<string>()` or `foo = input.required<string>()` |
| `@Output() saved = new EventEmitter()` | `saved = output<T>()` |
| `@ViewChild('x') x!: ElementRef` | `x = viewChild<ElementRef>('x')` |
| `constructor(private svc: Svc) {}` | `private svc = inject(Svc)` |
| `*ngIf="cond"` | `@if (cond) { ... }` |
| `*ngFor="let x of list"` | `@for (x of list; track x.id) { ... }` |
| `[ngSwitch]` | `@switch (value) { @case (...) { ... } }` |
| `BehaviorSubject` for local state | `signal()` |
| `subscribe()` in components | `httpResource` / `rxResource` / `toSignal` |
| `subscribe()` without `takeUntilDestroyed()` | always use `takeUntilDestroyed()` |
| Default change detection | `changeDetection: ChangeDetectionStrategy.OnPush` |
| Inline `template: ` longer than 5 lines | Separate `.html` file |
| `any` | Proper type, `unknown` if truly unknown |
| `as any` casts | Type guards, generics, or fix the actual type |
| Barrel `index.ts` re-exports in feature folders | Direct imports — barrels break tree-shaking |

---

## Component template

When creating a new component, use this skeleton:

```ts
import { ChangeDetectionStrategy, Component, computed, inject, input, output } from '@angular/core';
import { SomeService } from './some.service';

@Component({
  selector: 'app-thing',
  templateUrl: './thing.component.html',
  styleUrl: './thing.component.scss',
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class ThingComponent {
  private readonly svc = inject(SomeService);

  readonly id = input.required<string>();
  readonly variant = input<'primary' | 'secondary'>('primary');
  readonly saved = output<Thing>();

  readonly data = this.svc.getById(this.id);          // returns Signal<T> / Resource<T>
  readonly title = computed(() => this.data()?.name ?? '—');

  onSave(value: Thing): void {
    this.saved.emit(value);
  }
}
```

---

## Service template

```ts
import { httpResource } from '@angular/common/http';
import { inject, Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';

@Injectable({ providedIn: 'root' })
export class ThingService {
  private readonly http = inject(HttpClient);

  getById(id: Signal<string>) {
    return httpResource<Thing>(() => `/api/things/${id()}`);
  }

  save(thing: Thing) {
    return this.http.post<Thing>('/api/things', thing);
  }
}
```

---

## Routing template

```ts
import { Routes } from '@angular/router';

export default [
  {
    path: '',
    loadComponent: () => import('./pages/thing-list.page').then(m => m.ThingListPage),
  },
  {
    path: ':id',
    loadComponent: () => import('./pages/thing-detail.page').then(m => m.ThingDetailPage),
    resolve: { thing: thingResolver },
  },
] satisfies Routes;
```

---

## Tooling guarantees

- Don't introduce new dependencies without explicit approval. If you suggest one,
  call it out in the response.
- Prefer existing utilities in `core/` and `shared/`. Search before creating.
- Match existing patterns from the codebase even if they differ from these
  instructions. If you spot a violation of these instructions, mention it but
  do not silently refactor unrelated code.

---

## Comments & docs

- No JSDoc on private members.
- JSDoc only on exported services / public APIs / non-obvious utilities.
- No inline comments explaining WHAT — only WHY when non-obvious.
- No `// added for X`, `// removed Y`, `// TODO` without a ticket reference.

---

## Performance defaults

- `OnPush` always.
- `track` expression mandatory in `@for` — prefer stable IDs over `$index`.
- Lazy-load routes by default. Eager-load only the shell.
- `@defer (on viewport)` for heavy widgets, `@defer (on idle)` for low-priority.
- Images: `NgOptimizedImage` directive.
- Avoid `effect()` for data flow — `computed()` + template binding is enough 90% of the time.

---

## Accessibility (non-negotiable)

- Semantic HTML first; ARIA only when no semantic equivalent exists.
- Every interactive element keyboard-reachable; focus visible.
- Forms: label + `for` association, `aria-describedby` for hints/errors.
- Color contrast WCAG AA minimum.
