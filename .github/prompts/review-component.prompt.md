---
mode: agent
description: Review a component / file against modern Angular conventions
---

Review the selected file (or the file under cursor) against this project's conventions
defined in `copilot-instructions.md` and `instructions/*.md`.

## Report format

Produce a **structured review** with these sections. Don't change any code — just report.

### 1. Verdict
One of: ✅ Pass / ⚠ Minor issues / ❌ Significant issues. One-sentence summary.

### 2. Compliance check
Run through this checklist and mark each:

- [ ] Standalone (not in NgModule)
- [ ] `OnPush` change detection
- [ ] Separate `.html` + `.scss` files (no inline beyond 5 lines)
- [ ] `inject()` for all DI (no constructor params)
- [ ] Signal inputs (`input()`, `input.required()`)
- [ ] Signal outputs (`output()`)
- [ ] Signal queries (`viewChild()`, `viewChildren()`)
- [ ] New control flow in template (`@if`, `@for`, `@switch`)
- [ ] `@for` has `track` expression
- [ ] No `BehaviorSubject` for local state
- [ ] No `subscribe()` without `takeUntilDestroyed()`
- [ ] No `any` types
- [ ] Imports are tight (no unused)
- [ ] Has co-located `*.spec.ts`
- [ ] Accessibility: labels, ARIA, keyboard, focus

### 3. Findings
For each issue found:

- **File:line** — issue description
- **Severity**: blocker / major / minor / nit
- **Suggested fix**: one-line description

### 4. Performance opportunities
- Unnecessary `effect()` that could be `computed()`?
- Heavy work in template (call → computation)?
- Missing `@defer` for below-the-fold content?
- Missing `NgOptimizedImage`?
- Missing `track` in `@for`?

### 5. Refactoring opportunities (optional)
Only mention if they would meaningfully improve readability — don't suggest churn.

### 6. Test gaps
What public behavior isn't covered by the spec file?

## Do NOT
- Change any code in this review
- Be a pedant about style (Prettier handles that)
- Suggest renaming things just for taste
- Repeat the same finding for every occurrence — group them
