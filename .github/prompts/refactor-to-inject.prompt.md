---
mode: agent
description: Convert constructor DI to inject() function
---

Refactor the selected file (or files in the selected folder) from constructor-based DI
to **`inject()`-based DI**.

## Transformations

```ts
// BEFORE
export class ThingComponent {
  constructor(
    private readonly svc: ThingService,
    private readonly router: Router,
    @Inject(WINDOW) private readonly window: Window,
  ) {}
}

// AFTER
export class ThingComponent {
  private readonly svc = inject(ThingService);
  private readonly router = inject(Router);
  private readonly window = inject(WINDOW);
}
```

## Rules

- Field initializers must run in injection context (component/service/directive/pipe class body) — they do
- Mark all dependencies `private readonly` unless they're genuinely public
- Remove the constructor entirely IF its only job was DI
- Keep the constructor IF it has additional logic (initialization, super() call, etc.) — but still move DI to fields
- `@Inject(TOKEN)` becomes `inject(TOKEN)`
- `@Optional() dep: Dep | null` becomes `inject(Dep, { optional: true })`
- `@Self()`, `@SkipSelf()`, `@Host()` map to options: `inject(Dep, { self: true })` etc.
- Order: alphabetical by token name for consistency

## Edge cases

- **Class-based guards/resolvers/interceptors**: convert to functional (see `guards-resolvers.instructions.md`)
- **`ErrorHandler`** subclasses, `APP_INITIALIZER` factories that need DI — can still use `inject()` in their function body
- **Tests** using `TestBed.inject(...)` are unaffected — only production class DI changes

## Process

1. List all classes with constructor DI in the target scope
2. Convert one at a time
3. Verify no behavior change (no extra logic moved/lost)
4. Run TypeScript compile (or report what to run) to catch issues
