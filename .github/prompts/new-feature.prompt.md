---
mode: agent
description: Scaffold a complete feature folder (routes + pages + components + service + store + tests)
---

Scaffold a complete Angular feature folder following project conventions.

**Ask me first**:
1. Feature name (kebab-case, e.g. `customer-orders`)
2. Routes — list + detail? CRUD? custom?
3. Does it need a store?
4. Does it talk to an API? Endpoint base path?

**Then generate** this structure:

```
src/app/features/<feature>/
├── <feature>.routes.ts                   # default export Routes, lazy-loaded
├── <feature>.store.ts                    # signal store if needed
├── pages/
│   ├── <feature>-list.page.ts/html/scss + .spec.ts
│   └── <feature>-detail.page.ts/html/scss + .spec.ts
├── components/
│   └── <feature>-card.component.ts/html/scss + .spec.ts
├── services/
│   └── <feature>.service.ts + .spec.ts
└── models/
    └── <feature>.model.ts
```

**Rules**:
- All standalone, all OnPush
- Routes use `loadComponent`, `withComponentInputBinding` for params
- Service uses `httpResource` / `rxResource` for reads, `HttpClient` for writes
- Store uses NgRx Signal Store if installed, otherwise plain signal service
- Pages bind route params via `input.required()`
- Tests for every public class
- Add route to `app.routes.ts` (or relevant parent routes) with `loadChildren`

**Do not**:
- Create an NgModule
- Re-export via barrel `index.ts`
- Use any deprecated pattern from `copilot-instructions.md`
