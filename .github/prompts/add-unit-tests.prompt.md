---
mode: agent
description: Generate unit tests for the selected class
---

Generate a complete `*.spec.ts` for the selected class (component / service / store / guard / pipe).

## Process

1. Read the target file fully — understand its inputs, outputs, side effects, dependencies
2. Identify the **public API** to test (skip private methods)
3. List the test cases you intend to write — wait for my approval if it's a complex class
4. Generate the spec file co-located with source (`thing.component.ts` → `thing.component.spec.ts`)

## Test coverage minimums

### Component
- Renders successfully with required inputs set
- Renders correctly across input variations (e.g. `compact: true` vs `false`)
- Emits outputs on user interactions
- Reacts to signal/input changes (`setInput`, then `detectChanges`)
- Handles loading / error / empty states
- Accessibility smoke: focusable interactive elements, proper roles

### Service
- Each public method called with realistic inputs
- HTTP calls verified via `HttpTestingController`
- Error responses handled correctly
- State exposed (signals) updates as expected

### Store
- Initial state matches definition
- Each action updates state correctly
- Computed selectors return correct derived values
- Async actions: loading flag toggles, success populates, error captures

### Guard / Resolver
- Allow case
- Deny case (return UrlTree / false)
- Edge cases (no token, expired token, etc.)

### Pipe
- Happy path transformation
- `null` / `undefined` input
- Edge values (empty string, 0, very large input, special chars)

## Conventions

- Framework: **[Jest / Karma+Jasmine — match project setup]**
- Use `provideHttpClientTesting()` + `HttpTestingController` for HTTP
- Use `fixture.componentRef.setInput('name', value)` for signal inputs
- Mock dependencies via `provide: Svc, useValue: { ... }` — never via `jest.mock` for Angular DI
- `describe` blocks: one per class. Nested `describe` for grouping related tests.
- Test names start with verb: `it('emits saved on submit', ...)`
- AAA pattern with blank lines

## Do NOT
- Snapshot test HTML output
- Test framework internals (Angular CD, RxJS operators)
- Use `xit` / `fit` / `xdescribe` / `fdescribe` in committed code
- Write tests that pass without asserting anything meaningful
