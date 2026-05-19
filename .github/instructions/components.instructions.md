---
applyTo: "**/*.component.ts,**/*.page.ts"
description: Standalone Angular component rules (v20/v21)
---

# Component rules

When editing or generating a component class:

## Required decorator config
- `standalone: true` is implicit in v19+, but if codebase still sets it explicitly, follow that convention
- `changeDetection: ChangeDetectionStrategy.OnPush` — ALWAYS
- `templateUrl` + `styleUrl` — separate files, never inline `template:` / `styles:` beyond 5 lines
- `selector` follows `app-<kebab>` (or project prefix)
- `imports: []` only lists what the template actually uses — keep tight

## Component API
- Inputs: `readonly foo = input<T>()` or `input.required<T>()`
- Outputs: `readonly saved = output<T>()`
- Two-way: `readonly value = model<T>()` (template: `[(value)]="parentSignal"`)
- Queries: `readonly child = viewChild<ChildCmp>(ChildCmp)` (signal-based)
- Never use `@Input()`, `@Output()`, `@ViewChild()` decorators

## State inside the component
- Local state: `signal<T>()`
- Derived state: `computed(() => ...)` — pure functions, no side effects
- Async data: `httpResource(() => url)` or `rxResource({ request, loader })`
- Manual subscriptions: only with `takeUntilDestroyed()` (inject in field initializer)

## DI
```ts
private readonly svc = inject(SomeService);
private readonly router = inject(Router);
private readonly destroyRef = inject(DestroyRef);
```
Never use constructor injection.

## Lifecycle
- Avoid `ngOnInit` if `effect()` or `computed()` would do
- Use `afterNextRender` / `afterRender` for DOM measurements (zoneless-safe)
- Use `DestroyRef.onDestroy(() => ...)` instead of `ngOnDestroy` when convenient

## Forbidden in components
- `BehaviorSubject` for local state
- Direct DOM access via `ElementRef.nativeElement` (use Renderer2 or signal queries)
- `setTimeout` / `setInterval` outside of `afterRender`
- Imperative router navigation without typed params

## Skeleton

```ts
import { ChangeDetectionStrategy, Component, computed, inject, input, output } from '@angular/core';
import { ThingService } from '../services/thing.service';
import { Thing } from '../models/thing.model';

@Component({
  selector: 'app-thing-card',
  templateUrl: './thing-card.component.html',
  styleUrl: './thing-card.component.scss',
  changeDetection: ChangeDetectionStrategy.OnPush,
  imports: [],
})
export class ThingCardComponent {
  private readonly svc = inject(ThingService);

  readonly id = input.required<string>();
  readonly compact = input<boolean>(false);
  readonly opened = output<Thing>();

  readonly thing = this.svc.getById(this.id);
  readonly displayName = computed(() => {
    const t = this.thing.value();
    return t ? `${t.name} (${t.code})` : '—';
  });
}
```
