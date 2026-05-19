---
applyTo: "**/*.guard.ts,**/*.resolver.ts,**/*.interceptor.ts"
description: Functional guards, resolvers, interceptors
---

# Guards, resolvers, interceptors — functional only

Class-based guards/resolvers/interceptors are deprecated. Use functional APIs.

## Guard

```ts
import { CanActivateFn, Router } from '@angular/router';
import { inject } from '@angular/core';
import { AuthService } from '../services/auth.service';

export const authGuard: CanActivateFn = (route, state) => {
  const auth = inject(AuthService);
  const router = inject(Router);

  if (auth.isAuthenticated()) return true;

  return router.createUrlTree(['/login'], {
    queryParams: { returnUrl: state.url },
  });
};
```

## CanMatch (preferred over CanActivate for lazy routes)

```ts
export const adminMatch: CanMatchFn = () => {
  return inject(AuthService).hasRole('admin');
};
```

## Resolver

```ts
import { ResolveFn } from '@angular/router';
import { inject } from '@angular/core';
import { ThingService } from './thing.service';

export const thingResolver: ResolveFn<Thing> = (route) => {
  const id = route.paramMap.get('id')!;
  return inject(ThingService).fetchById(id);
};
```

## HTTP Interceptor

```ts
import { HttpInterceptorFn } from '@angular/common/http';
import { inject } from '@angular/core';
import { AuthService } from '../services/auth.service';

export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const token = inject(AuthService).token();
  if (!token) return next(req);

  return next(req.clone({
    setHeaders: { Authorization: `Bearer ${token}` },
  }));
};
```

Register in `app.config.ts`:

```ts
provideHttpClient(withInterceptors([authInterceptor, errorInterceptor]))
```

## Rules
- One guard/resolver/interceptor per file
- Always use `inject()` inside the function
- Return `boolean | UrlTree | Observable<boolean | UrlTree> | Promise<...>`
- Prefer `CanMatch` over `CanActivate` for route guarding (works with lazy loading + avoids loading the chunk)
- Never inject `ActivatedRoute` in resolvers — use the `route` parameter
