---
mode: agent
description: Convert RxJS / BehaviorSubject / decorator-based component to signal-based
---

Convert the selected file (or files in the selected folder) from RxJS-heavy / decorator-based
Angular code to **signal-based** Angular 20+ code.

## Conversion rules

| Before | After |
|---|---|
| `@Input() foo: string` | `readonly foo = input<string>()` |
| `@Input({ required: true }) foo!: string` | `readonly foo = input.required<string>()` |
| `@Output() saved = new EventEmitter<T>()` | `readonly saved = output<T>()` |
| `@ViewChild('x') x!: ElementRef` | `readonly x = viewChild<ElementRef>('x')` |
| `@ViewChildren(Cmp) items!: QueryList<Cmp>` | `readonly items = viewChildren(Cmp)` |
| `constructor(private svc: Svc) {}` | `private readonly svc = inject(Svc);` |
| `subject = new BehaviorSubject<T>(initial)` | `subject = signal<T>(initial)` |
| `observable$ \| async` in template | `observable()` (after `toSignal()` or `httpResource`) |
| Manual `subscribe()` in component | `httpResource` / `rxResource` / `toSignal({ initialValue })` |
| `ngOnChanges` reacting to input | `effect(() => ...)` or `computed(() => ...)` |
| `combineLatest([a$, b$])` for derived | `computed(() => fn(a(), b()))` |

## Template updates

- Replace `(observable$ | async)` with `signal()` calls
- Replace `*ngIf="x$ | async as x"` with `@if (x(); as x) { ... }`
- Remove `| async` pipes — signals are called directly

## Subscription management

Any remaining `subscribe()` calls MUST use `takeUntilDestroyed()`:

```ts
private readonly destroyRef = inject(DestroyRef);

ngOnInit() {
  this.svc.events$.pipe(takeUntilDestroyed(this.destroyRef)).subscribe(...);
}
```

## Process

1. Read the target file completely first
2. Identify every transformation needed (list them in chat)
3. Apply transformations, keeping behavior identical
4. Update the template to match (call signals as functions, replace `*ngIf`/`*ngFor`)
5. Update the `.spec.ts` if input/output API changed
6. If any RxJS operator chain is non-trivial (>3 operators), ask before converting
7. Report what was converted and what was left untouched and why
