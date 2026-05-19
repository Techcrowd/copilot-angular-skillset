---
mode: agent
description: Create a new standalone Angular component following project conventions
---

Create a new standalone Angular component with the following requirements.

**Ask me first** (if not provided in the original prompt):
1. Component name (PascalCase, e.g. `UserProfileCard`)
2. Where it lives (e.g. `features/users/components/`)
3. Inputs (name + type + required?)
4. Outputs (name + payload type)
5. Does it need any service?

**Then generate**:

1. `*.component.ts` with:
   - `standalone: true`
   - `changeDetection: OnPush`
   - separate `templateUrl` + `styleUrl`
   - `inject()` for all DI
   - `input()` / `input.required()` for inputs
   - `output()` for outputs
   - `computed()` for derived state
2. `*.component.html` with:
   - new control flow (`@if`, `@for`, `@switch`)
   - semantic HTML
   - accessible (labels, ARIA where needed)
3. `*.component.scss` minimal — host `display: block`, only what's needed
4. `*.component.spec.ts` with:
   - rendering smoke test
   - input change test
   - output emission test

**Conventions** (do not violate):
- No `NgModule`, no decorator-based `@Input/@Output/@ViewChild`
- No `*ngIf` / `*ngFor` / `*ngSwitch`
- No constructor injection
- No `BehaviorSubject` for local state — use `signal()`
- No inline templates / styles
- Match existing folder structure and naming
