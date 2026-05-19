---
applyTo: "**/*.store.ts,**/+state/**/*.ts,**/state/**/*.ts"
description: State management — NgRx Signal Store / signals-only stores
---

# Store rules

This project uses **signals-based state management**. If NgRx Signal Store is
installed, prefer it for feature stores. Otherwise use plain signal services.

## NgRx Signal Store template

```ts
import { signalStore, withState, withComputed, withMethods, patchState } from '@ngrx/signals';
import { computed, inject } from '@angular/core';
import { ThingService } from './thing.service';

type ThingsState = {
  items: Thing[];
  filter: string;
  loading: boolean;
  error: string | null;
};

const initialState: ThingsState = {
  items: [],
  filter: '',
  loading: false,
  error: null,
};

export const ThingsStore = signalStore(
  { providedIn: 'root' },
  withState(initialState),
  withComputed(({ items, filter }) => ({
    filtered: computed(() =>
      items().filter(i => i.name.toLowerCase().includes(filter().toLowerCase()))
    ),
    count: computed(() => items().length),
  })),
  withMethods((store, svc = inject(ThingService)) => ({
    async load() {
      patchState(store, { loading: true, error: null });
      try {
        const items = await svc.fetchAll();
        patchState(store, { items, loading: false });
      } catch (e) {
        patchState(store, { error: String(e), loading: false });
      }
    },
    setFilter(filter: string) {
      patchState(store, { filter });
    },
  })),
);
```

## Plain signal store (no NgRx dependency)

```ts
@Injectable({ providedIn: 'root' })
export class ThingsStore {
  private readonly svc = inject(ThingService);

  private readonly _items = signal<Thing[]>([]);
  private readonly _filter = signal('');
  private readonly _loading = signal(false);

  readonly items = this._items.asReadonly();
  readonly filter = this._filter.asReadonly();
  readonly loading = this._loading.asReadonly();
  readonly filtered = computed(() =>
    this._items().filter(i =>
      i.name.toLowerCase().includes(this._filter().toLowerCase())
    )
  );

  setFilter(f: string) { this._filter.set(f); }

  async load() {
    this._loading.set(true);
    try {
      this._items.set(await this.svc.fetchAll());
    } finally {
      this._loading.set(false);
    }
  }
}
```

## Rules
- State is **deeply immutable** — replace, don't mutate
- Selectors are `computed()` — pure, no side effects
- Async actions live in methods, not in `effect()`
- One store per feature; cross-feature reads via injection of the other store
- Avoid `effect()` for chained state changes — use `linkedSignal()` or `computed()`

## What to avoid
- NgRx Store (classic) — only Signal Store in this project
- Circular store dependencies — refactor to a shared parent store
- Storing UI-only state (open/closed, hover) in a global store — keep in component
