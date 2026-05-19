---
mode: agent
description: Upgrade the project from Angular 20 to Angular 21
---

Upgrade this project from Angular 20 to Angular 21. Work methodically — don't skip steps.

## Pre-upgrade audit

Before changing anything, report:

1. **Current version**: read `package.json` — confirm `@angular/core`, `@angular/cli`, `typescript`, `rxjs` versions
2. **Build status**: is `ng build` currently green? If not, fix before upgrading
3. **Test status**: is the test suite green?
4. **Outdated patterns** that v21 enforces / removes:
   - Any remaining `NgModule`? Count.
   - Any `*ngIf` / `*ngFor` / `*ngSwitch`? Count.
   - Any `@Input()` / `@Output()` decorators? Count.
   - Any class-based guards/resolvers/interceptors? Count.
5. **Third-party libs** to upgrade alongside:
   - `@angular/material`, `@angular/cdk`
   - `@ngrx/*` (if used)
   - `@ng-bootstrap/*`, `ng-zorro-antd`, `primeng`, etc.

Report this audit and **wait for my go-ahead** before proceeding.

## Upgrade process

1. **Use `ng update`** — never edit `package.json` by hand for Angular packages:
   ```
   ng update @angular/core@21 @angular/cli@21
   ```
2. **Then update peer libs**:
   ```
   ng update @angular/material@21
   ng update @ngrx/signals@latest
   ```
3. **Run schematics** that Angular ships:
   ```
   ng generate @angular/core:control-flow
   ng generate @angular/core:standalone
   ng generate @angular/core:inject
   ng generate @angular/core:signal-input-migration
   ng generate @angular/core:signal-queries-migration
   ng generate @angular/core:output-migration
   ```
   Run each, then review the diff, then commit.

4. **Manual cleanup** (schematics don't catch everything):
   - Remove leftover `NgModule` imports
   - Delete empty barrel `index.ts` files
   - Update test setup to remove deprecated APIs
   - Replace `HttpClientModule` import with `provideHttpClient()`
   - Replace `RouterModule.forRoot()` with `provideRouter()`

5. **Check zoneless**:
   - If considering zoneless, add `provideZonelessChangeDetection()` to `app.config.ts`
   - Remove `zone.js` from `polyfills` in `angular.json`
   - Run full test suite — fix any code that relied on Zone.js patching

6. **Verify**:
   - `ng build` green
   - `ng test` green
   - Manual smoke test of main flows
   - Check bundle size delta (should usually shrink)

## Reporting

After each step, report:
- What changed
- What failed and how you fixed it
- What needs my decision

## Do NOT
- Skip the audit
- Edit `package.json` versions by hand
- Combine multiple major migrations into one commit
- Disable strict TypeScript flags to make errors go away
- Suppress lint rules instead of fixing them
