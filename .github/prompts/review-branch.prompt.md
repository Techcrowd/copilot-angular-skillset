---
mode: agent
description: Review all changes on a branch vs main/master against project conventions
---

You are reviewing a feature branch against the project baseline (default `main`,
fallback `master`). Compare the diff against this project's conventions defined
in `.github/copilot-instructions.md` and `.github/instructions/*.md`.

## Inputs

If the user didn't provide them, **ask first**:
1. Branch to review (e.g. `feature/customer-orders`). If empty, use current branch.
2. Base branch (default: `main`; fallback `master` if `main` doesn't exist).
3. Scope hint (optional) — e.g. "focus on signals migration", "focus on a11y".

## Procedure

Run these in the integrated terminal — don't modify any code.

1. **Resolve refs**:
   ```bash
   git fetch --no-tags origin
   git rev-parse --verify <base>           # confirm base exists
   git rev-parse --verify <branch>         # confirm branch exists
   git merge-base <base> <branch>          # divergence point
   ```
2. **Collect changed files**:
   ```bash
   git diff --name-status <base>...<branch>
   git diff --stat <base>...<branch>
   git log --no-merges --oneline <base>..<branch>
   ```
3. **Read the diff** (chunk by file if large):
   ```bash
   git diff <base>...<branch> -- <path>
   ```
4. For each non-trivial changed file, **open the current version** to understand
   the surrounding context (don't review patches in isolation).
5. Cross-check each change against:
   - `.github/copilot-instructions.md` (global)
   - The matching `.github/instructions/*.md` by glob (components, templates,
     services, stores, routing, guards-resolvers, pipes-directives, styles, tests).

## Report format

Produce a **structured review** — do not change any code. Markdown only.

### 1. Summary
- **Branch**: `<branch>` → `<base>` (merge-base `<sha>`)
- **Changed files**: N (added X / modified Y / deleted Z / renamed R)
- **Commits**: N (squashable? meaningful messages?)
- **Verdict**: ✅ Ready to merge / ⚠ Needs follow-ups / ❌ Block — one-sentence rationale.

### 2. What changed (1-paragraph narrative)
Plain-English description of the intent of the branch — features added,
refactors performed, bugs fixed. Inferred from commits + diff, not guessed.

### 3. Per-area compliance
Group findings by area. Skip areas with no changes.

#### Components
- [ ] Standalone, `OnPush`, `inject()`, signal API, separate `.html`/`.scss`

#### Templates
- [ ] `@if` / `@for` (with `track`) / `@switch` / `@let` — no `*ngIf` / `*ngFor`
- [ ] No `ngTemplateOutlet` where `@if` works
- [ ] Accessibility: labels, ARIA, keyboard, focus visible

#### Services
- [ ] `inject()`, `providedIn: 'root'` when singleton
- [ ] `httpResource` / `rxResource` for reads where applicable
- [ ] No leaked `subscribe()` — `takeUntilDestroyed()` enforced

#### Stores / state
- [ ] Signals or chosen store lib — no ad-hoc `BehaviorSubject` for local state

#### Routing
- [ ] Lazy `loadComponent` / `loadChildren`, route-level `providers`, typed `Routes`

#### Guards / resolvers
- [ ] Functional guards/resolvers — no class-based legacy guards

#### Pipes / directives
- [ ] Standalone, pure where possible

#### Styles
- [ ] Follows chosen styling stack (Tailwind v4 / SCSS / Material) — no mixing

#### Tests
- [ ] Co-located `*.spec.ts` for new public classes
- [ ] Behavior covered, not implementation details

### 4. Findings
For each issue:
- **`<path>:<line>`** — short description
- **Severity**: 🛑 blocker / 🔴 major / 🟡 minor / ⚪ nit
- **Why it matters**: 1 sentence tied to a rule in `copilot-instructions.md`
- **Suggested fix**: one line — code snippet only when it's a clear one-liner

Group repeated occurrences into one finding ("12× in `pages/*.page.ts` —
constructor injection still used"). Do not list each file separately.

### 5. Cross-cutting concerns
Things visible only at branch-level (not in any single file):
- New dependencies (`package.json` diff) — justified? size impact?
- Public API changes (exports renamed/removed) — breaking?
- Migration consistency — half-migrated areas (some files on signals, some not)?
- Bundle/perf risk — eager imports that should be `@defer` / lazy?
- Feature flags / env config — added but not wired?
- Generated files committed? (`dist/`, lockfiles diverged?)

### 6. Test coverage gaps
What new public behavior on the branch has no spec? Reference the file.

### 7. Risk & merge readiness
- Conflicts likely with `<base>`? (use `git merge-tree` if useful)
- Reversibility — can this be safely reverted as one commit?
- Anything that needs a follow-up ticket?

## Do NOT
- Modify any code, run `git commit`, `git push`, or `git rebase`.
- Comment on Prettier-level formatting.
- Suggest churn-only refactors ("rename for taste").
- Repeat the same finding per file — group them.
- Approve changes that violate hard anti-patterns in `copilot-instructions.md`
  without flagging them as blockers.
