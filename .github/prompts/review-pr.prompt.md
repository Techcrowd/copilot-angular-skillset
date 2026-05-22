---
mode: agent
description: Review a GitHub pull request against project conventions
---

You are reviewing a GitHub pull request against this project's conventions defined
in `.github/copilot-instructions.md` and `.github/instructions/*.md`.

## Inputs

If not provided, **ask first**:
1. PR number or URL (e.g. `#142` or `https://github.com/<org>/<repo>/pull/142`).
2. Scope hint (optional).

## Procedure

Use the GitHub CLI (`gh`) if available; otherwise fall back to git on the
checked-out branch. Don't modify any code.

1. **Fetch PR metadata**:
   ```bash
   gh pr view <num> --json title,body,baseRefName,headRefName,author,additions,deletions,changedFiles,commits
   ```
2. **Get the diff** (prefer file list + per-file diff over one giant blob):
   ```bash
   gh pr diff <num> --name-only
   gh pr diff <num> -- <path>
   ```
3. **Read the PR description** — confirm the diff matches the stated intent.
4. For each non-trivial file, open the current version on the head branch to
   understand surrounding context (don't review patches in isolation).
5. Cross-check each change against the matching `.github/instructions/*.md`.

If `gh` is unavailable, follow `/review-branch` with `head = <head ref>`
and `base = <base ref>` from the PR.

## Report format

Same structured report as `/review-branch`, plus at the top:

### 0. PR header
- **Title**: …
- **Author**: …
- **Base ← Head**: `<base>` ← `<head>`
- **Size**: +X / −Y across N files / M commits
- **Stated intent (from description)**: 1-line summary
- **Does the diff match the intent?**: yes / partially / no — why

Then continue with sections 1–7 from `/review-branch`:
Summary, What changed, Per-area compliance, Findings, Cross-cutting concerns,
Test coverage gaps, Risk & merge readiness.

## Do NOT
- Post comments to GitHub (`gh pr comment`, `gh pr review`) unless the user
  explicitly asks for it. Default = local markdown report.
- Approve or merge the PR.
- Modify any code on the head branch.
- Repeat the same finding per file — group them.
