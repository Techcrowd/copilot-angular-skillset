---
applyTo: "**/*.service.ts"
description: Angular service rules — DI, HTTP, signals, resources
---

# Service rules

## Provisioning
- Default: `@Injectable({ providedIn: 'root' })` for singletons
- Feature-scoped: provide in route `providers: []`
- Component-scoped: provide in component `providers: []` (rare)
- Never declare services in NgModules — they don't exist in this project

## DI
- Always `inject()` in field initializers — never constructor params
- Mark deps `private readonly`
- Don't inject `ChangeDetectorRef` in services (services shouldn't trigger CD)

## HTTP
- Prefer `httpResource<T>(() => url, options?)` for reactive GET
- Use `HttpClient` directly only for mutations (POST/PUT/PATCH/DELETE)
- Always type the response: `http.post<Thing>(url, body)`
- Errors: let interceptors handle global cases; throw or return error state for caller-specific handling
- Never store the `HttpClient` subscription — return the Observable, let caller use `rxResource` or `firstValueFrom`

## Signals in services
- Expose state as `Signal<T>` or `readonly` `WritableSignal<T>` (callers cannot mutate)
- For derived state expose `computed()`
- For async data expose `Resource<T>` (`httpResource` / `rxResource`)
- Don't expose `BehaviorSubject` — wrap in `toSignal()` or convert fully

## Skeleton — read-mostly service

```ts
import { httpResource, HttpClient } from '@angular/common/http';
import { inject, Injectable, signal, Signal } from '@angular/core';

@Injectable({ providedIn: 'root' })
export class ThingService {
  private readonly http = inject(HttpClient);

  getById(id: Signal<string>) {
    return httpResource<Thing>(() => `/api/things/${id()}`);
  }

  list(filter: Signal<ThingFilter>) {
    return httpResource<Thing[]>(() => ({
      url: '/api/things',
      params: filter(),
    }));
  }

  async create(thing: ThingDraft): Promise<Thing> {
    return firstValueFrom(this.http.post<Thing>('/api/things', thing));
  }
}
```

## Skeleton — stateful service (cache + invalidation)

```ts
@Injectable({ providedIn: 'root' })
export class ThingsStore {
  private readonly http = inject(HttpClient);
  private readonly _items = signal<Thing[]>([]);
  private readonly _loading = signal(false);

  readonly items = this._items.asReadonly();
  readonly loading = this._loading.asReadonly();
  readonly count = computed(() => this._items().length);

  async refresh(): Promise<void> {
    this._loading.set(true);
    try {
      const data = await firstValueFrom(this.http.get<Thing[]>('/api/things'));
      this._items.set(data);
    } finally {
      this._loading.set(false);
    }
  }
}
```

## What to avoid
- `Subject` / `BehaviorSubject` as public API — use signals
- `this.x$ = this.http.get(...)` patterns (cold observables stored as fields)
- Services that touch the DOM
- Services that import from `@angular/router` and trigger navigation without a clear reason — prefer letting components navigate
