---
mode: agent
description: Migrate an NgModule + its declarations to standalone components
---

Migrate the selected `*.module.ts` file (and its declared components/directives/pipes)
to **standalone** Angular APIs.

## Process

1. Read the NgModule file. List:
   - `declarations: []`
   - `imports: []`
   - `providers: []`
   - `exports: []`
2. For **each declared component / directive / pipe**:
   - Add `standalone: true` to the decorator
   - Move the needed `imports` from the NgModule into the component's `imports: []`
   - If the NgModule imported `CommonModule`, replace with the specific items the
     template actually uses (or nothing if using new control flow)
3. For **each provider in `providers: []`**:
   - If `providedIn: 'root'` already in the service, remove from NgModule (it's redundant)
   - Otherwise move to:
     - `app.config.ts` if app-wide
     - Route `providers: []` if feature-scoped
     - Component `providers: []` if component-scoped
4. For **routing modules** (`RouterModule.forChild(routes)` / `forRoot`):
   - Convert the routes array into a separate `<feature>.routes.ts` exporting `default`
   - Update parent route to use `loadChildren: () => import('./feature.routes')`
5. For **`exports: []`**:
   - Standalone items are imported directly from their file — no re-export needed
   - Delete the barrel `index.ts` if it only re-exported NgModule contents
6. **Delete the NgModule file**
7. Update any place that imported the NgModule:
   - `imports: [FeatureModule]` → import the specific standalone components/directives directly

## Common patterns

### NgModule with declarations

```ts
// BEFORE
@NgModule({
  declarations: [UserCardComponent, UserBadgeComponent],
  imports: [CommonModule, MatButtonModule],
  exports: [UserCardComponent],
})
export class UsersModule {}
```

After: `UserCardComponent` becomes standalone with its own `imports: [MatButton]`,
`UserBadgeComponent` likewise. Module file deleted.

### Routing module

```ts
// BEFORE — users-routing.module.ts
@NgModule({
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule],
})
export class UsersRoutingModule {}
```

After: deleted. Routes live in `users.routes.ts`. Parent route uses `loadChildren`.

### Shared module

If a "SharedModule" re-exports common components/pipes, just import each item
directly where needed. Delete the shared module.

## Safety

- After migration, run `ng build` (or report what to run) to catch missed references
- Update `*.spec.ts` files — they likely import the NgModule; replace with direct standalone imports
- If a third-party library only exposes NgModules, keep importing them in the standalone component's `imports: []`

## Do NOT
- Mix NgModule and standalone in the same file
- Use `importProvidersFrom()` unless the library genuinely has no standalone API
- Re-create barrel `index.ts` files
