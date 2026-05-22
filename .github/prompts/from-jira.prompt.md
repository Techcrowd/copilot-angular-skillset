---
mode: agent
description: Fetch a Jira issue and propose an implementation plan (no code changes)
---

You are reading a Jira issue and proposing an implementation plan. **You do NOT
write any production code in this prompt** — you stop after producing the plan
and wait for the user to approve / redirect / call a follow-up prompt.

## Inputs

If not provided, **ask first**:
1. Jira issue — key (`PROJ-123`) or full URL.
2. (Optional) Branch you'll work on. If omitted, suggest one based on issue key + title.

## Required environment

The host machine must expose these (check via `env | grep -E '^JIRA_'` — never
echo the token to chat or terminal output):

- `JIRA_BASE_URL` — e.g. `https://acme.atlassian.net`
- `JIRA_EMAIL` — the email tied to the API token
- `JIRA_API_TOKEN` — Atlassian API token (id.atlassian.com → Security → API tokens)

If any is missing, **stop and tell the user** which one — link to `AGENTS.md`
in the repo root for setup. Do not prompt them to paste the token into chat.

## Procedure

Run in the integrated terminal. Never write the token into commands you echo
back — use `$JIRA_API_TOKEN` references only.

### 1. Resolve the issue key

If user gave a URL like `https://acme.atlassian.net/browse/PROJ-123`, extract
`PROJ-123`. If they gave a key, use it directly. Also derive `JIRA_BASE_URL`
from the URL if user provided it — don't hardcode anything in this prompt.

### 2. Fetch the issue (REST v3)

Prefer the official Atlassian CLI if installed (`command -v acli` returns a path):

```bash
acli jira workitem view <KEY> --json
```

Otherwise fall back to `curl` + REST v3:

```bash
curl -s -u "$JIRA_EMAIL:$JIRA_API_TOKEN" \
  -H "Accept: application/json" \
  "$JIRA_BASE_URL/rest/api/3/issue/<KEY>?fields=summary,description,status,issuetype,priority,assignee,labels,components,fixVersions,parent,subtasks,issuelinks,attachment,comment,customfield_10016&expand=renderedFields"
```

Pipe through `jq` if available, otherwise read JSON directly.

### 3. Fetch comments (if not already inlined)

```bash
curl -s -u "$JIRA_EMAIL:$JIRA_API_TOKEN" \
  -H "Accept: application/json" \
  "$JIRA_BASE_URL/rest/api/3/issue/<KEY>/comment"
```

### 4. Render description + comments

Jira returns Atlassian Document Format (ADF) in `description` and
`comment[].body`. `renderedFields.description` gives you HTML — convert to
plain markdown by stripping HTML tags. Don't try to be fancy with ADF parsing.

## Report format

Produce exactly this structure as markdown. **No code edits in this prompt.**

### 1. Header
- **Key**: `PROJ-123` — [title]
- **Type / Status / Priority**: …
- **Assignee / Reporter**: …
- **Sprint / Fix version / Components / Labels**: …
- **Parent / Subtasks / Linked issues**: bulleted list with keys + titles
- **Attachments**: filename list (don't download)
- **Link**: `$JIRA_BASE_URL/browse/PROJ-123`

### 2. What is being asked (1 paragraph)
Plain-English restatement of the goal — distilled from description +
comments. Resolve contradictions (e.g. description says X but later comment
says Y) by flagging them, not by picking silently.

### 3. Acceptance criteria
Extract verbatim if structured (checkbox list). If buried in prose, summarize
as a checklist and **mark items you inferred** vs ones written explicitly.

### 4. Open questions
Things that block confident implementation. Each item:
- The question
- Who should answer (assignee / reporter / tech lead)
- What you'll assume if no answer is given

### 5. Implementation plan
- **Affected areas**: feature folders / files (best guess from description +
  current repo state — run `git grep` / file search to ground this).
- **Suggested branch name**: `<type>/<KEY>-<kebab-summary>`.
- **Steps**: ordered list. For each step, name the existing prompt to call:
  - `/new-component` — when a new standalone component is needed
  - `/new-feature` — when a whole feature folder is needed
  - `/convert-to-signals`, `/migrate-to-control-flow`,
    `/migrate-ngmodule-to-standalone`, `/refactor-to-inject` — for migration tasks
  - `/add-unit-tests` — after touching public classes
  - `/fix-onpush-issues` — if change detection might break
  - `/review-component` — file-level QA
  - `/review-branch` — before opening PR
- **Estimated touch points**: rough count of files (3 / 5–10 / 10+).
- **Risks**: bullet list — public API breaks, perf regressions, a11y impact,
  migration half-done elsewhere, anything that conflicts with
  `copilot-instructions.md`.
- **Out of scope** (explicit): things the ticket implies but you'll **not** do.

### 6. Next action
One sentence: "Approve this plan and I'll start with `/new-feature` for
`features/<x>`" — or whichever prompt fits. **Stop here.**

## Do NOT
- Write or modify any code in this prompt.
- Commit, push, branch, or run destructive git.
- Post a comment back to Jira (no `acli jira workitem comment` etc.).
- Echo `$JIRA_API_TOKEN` into terminal output, the chat, or any file.
- Make assumptions about state-mgmt / styling / testing stack — pull those from
  `.github/copilot-instructions.md` of the current repo. Skillset is generic.
- Implement until the user explicitly says "go" / "do it" / "proceed" or names
  the follow-up prompt to run.
