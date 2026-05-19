---
applyTo: "**/*.routes.ts,**/app.routes.ts,**/app.config.ts"
description: Routing — lazy load, functional guards, typed params
---

# Routing rules

## Always lazy-load feature routes

```ts
// app.routes.ts
import { Routes } from '@angular/router';

export const appRoutes: Routes = [
  { path: '', pathMatch: 'full', redirectTo: 'dashboard' },
  {
    path: 'dashboard',
    loadComponent: () => import('./features/dashboard/dashboard.page').then(m => m.DashboardPage),
  },
  {
    path: 'things',
    loadChildren: () => import('./features/things/things.routes'),
  },
  {
    path: '**',
    loadComponent: () => import('./features/not-found/not-found.page').then(m => m.NotFoundPage),
  },
];
```

## Feature routes file

```ts
// features/things/things.routes.ts
import { Routes } from '@angular/router';
import { authGuard } from '../../core/guards/auth.guard';
import { thingResolver } from './services/thing.resolver';

export default [
  {
    path: '',
    loadComponent: () => import('./pages/thing-list.page').then(m => m.ThingListPage),
    canActivate: [authGuard],
  },
  {
    path: ':id',
    loadComponent: () => import('./pages/thing-detail.page').then(m => m.ThingDetailPage),
    resolve: { thing: thingResolver },
    providers: [/* feature-scoped */],
  },
] satisfies Routes;
```

## App config

```ts
// app.config.ts
import { ApplicationConfig, provideZonelessChangeDetection } from '@angular/core';
import { provideRouter, withComponentInputBinding, withViewTransitions } from '@angular/router';
import { provideHttpClient, withInterceptors, withFetch } from '@angular/common/http';

export const appConfig: ApplicationConfig = {
  providers: [
    provideZonelessChangeDetection(),               // when ready for zoneless
    provideRouter(appRoutes, withComponentInputBinding(), withViewTransitions()),
    provideHttpClient(withFetch(), withInterceptors([authInterceptor, errorInterceptor])),
  ],
};
```

## Param binding — use `withComponentInputBinding()`

```ts
// in the page component
readonly id = input.required<string>();   // bound from route param :id
```

No `ActivatedRoute` subscription needed.

## Guards & resolvers
- **Functional only** — `CanActivateFn`, `CanMatchFn`, `ResolveFn`
- Never class-based guards/resolvers (deprecated)
- Inject deps with `inject()` inside the function

## Rules
- One feature = one `*.routes.ts` exporting `default` as `Routes`
- Use `loadComponent` for single-page features, `loadChildren` for nested routes
- Use `pathMatch: 'full'` only when matching empty path
- Avoid redirect chains; prefer one direct redirect
- Don't use `RouterModule.forRoot()` / `forChild()` — they're NgModule patterns

## What to avoid
- Eager-loaded feature routes (only shell is eager)
- Subscribing to `route.params` in components — use `withComponentInputBinding` + `input()`
- Imperative navigation strings — use typed route helpers if a `Routes`-typed config exists
