---
name: init-code
description: >
  Scaffold the code workspace for an already-initialized ideation project. Use when the
  user says "init-code", "set up the code workspace", "scaffold /code/", "promote this
  project to code", or otherwise asks to create the `/code/` subtree (git repo, code
  wiki, code-mode CLAUDE.md). Triggers on first run (no `/code/` exists) and on re-runs
  (`/code/` already exists, possibly already a git repo from a prior clone — additive
  scaffold only).
---

# Code Workspace Init

When triggered, execute all steps autonomously. The user's selected folder is the **ideation project root** — the parent of `/code/`, not `/code/` itself. Refuse and abort if there is no `CLAUDE.md` at the project root: `init-code` is an extension to an already-initialized project, not a replacement for `init`.

This skill supports two modes:

- **First run** — `/code/` does not exist. Create it, run `git init`, scaffold all code-side files (CLAUDE.md, README.md, DECISIONS.md, .gitignore, code wiki), make a single first commit (`init: scaffold code workspace`).
- **Re-run** — `/code/` already exists. Detect whether it is already a git repo (`/code/.git/` present). If yes, skip `git init` and skip the first commit; just additively scaffold any missing wiki pieces or templated files. If no, run `git init` but still treat existing files as authoritative (additive merge for code-mode CLAUDE.md, create-if-missing for everything else).

Re-runs are **strictly additive** outside the deprecated-sections channel — same discipline as ideation `init`.

---

## Step 1 — Verify and detect

1. Confirm `CLAUDE.md` exists at the project root. If not, abort with: *"`init-code` requires an already-initialized ideation project. Run `/yuval-core:init` first."*
2. Detect:
   - **PROJECT_NAME** — basename of the project root.
   - **DATE** — today's date in `YYYY-MM-DD`.
   - **CODE_EXISTS** — does `/code/` exist?
   - **GIT_EXISTS** — does `/code/.git/` exist?
3. Set MODE based on detection:
   - `CODE_EXISTS=false` → MODE = first-run.
   - `CODE_EXISTS=true, GIT_EXISTS=true` → MODE = re-run-existing-repo (the workbench / cloned-repo case).
   - `CODE_EXISTS=true, GIT_EXISTS=false` → MODE = re-run-no-repo (rare; treat additively, run `git init`).

## Step 2 — Read the bundled references

Five files are bundled with this skill in `references/`:

- `${SKILL_DIR}/references/code-claude-template.md` — code-mode CLAUDE.md template
- `${SKILL_DIR}/references/code-readme-template.md` — initial README.md
- `${SKILL_DIR}/references/code-decisions-template.md` — initial DECISIONS.md
- `${SKILL_DIR}/references/code-gitignore-template` — initial .gitignore
- `${SKILL_DIR}/references/code-wiki-templates/` directory containing:
  - `memory-protocol.md` — read from `../../init/references/memory-protocol.md` to keep ideation and code copies identical (single source of truth at the plugin level)
  - `index.md` — code-wiki index template
  - `log.md` — code-wiki log template
  - `next.md` — code-wiki next template

Read all of them via the Read tool. Missing files indicate a broken plugin install — warn and stop.

For `memory-protocol.md`: read from `${PLUGIN_ROOT}/skills/init/references/memory-protocol.md`. The code workspace and the ideation workspace share the canonical protocol — there is only one copy in the plugin bundle, populated from `skills/init/`. Do not duplicate the file in `init-code/references/`.

## Step 3 — Create `/code/` if missing

If MODE is first-run, `mkdir -p /code/`. Otherwise skip.

## Step 4 — Run `git init` if needed

- If MODE is first-run or re-run-no-repo: `cd /code && git init -b main`. Use `-b main` so the initial branch is `main` (matches D-015).
- If MODE is re-run-existing-repo: skip. Do not touch git state. The user's existing remote, branches, and tags are untouched.

## Step 5 — Scaffold code-mode files

For each of the four files below, apply create-if-missing semantics — the same logic ideation `init` uses. **Do not overwrite an existing file** (the workbench's `/code/README.md`, for example, must survive).

### `/code/CLAUDE.md` — code-mode CLAUDE.md (additive merge if present)

- If absent: perform placeholder substitution (`{{PROJECT_NAME}}`, `{{DATE}}`) on `code-claude-template.md` and write to `/code/CLAUDE.md`.
- If present: apply the same additive-merge logic as ideation `init` Step 5 — section by section, append new sections or bullets from the template, never remove or shorten existing content.

### `/code/README.md` — create-if-missing

- If absent: substitute `{{PROJECT_NAME}}` in `code-readme-template.md`, write to `/code/README.md`.
- If present: leave alone.

### `/code/DECISIONS.md` — create-if-missing

- If absent: substitute `{{PROJECT_NAME}}` and `{{DATE}}` in `code-decisions-template.md`, write to `/code/DECISIONS.md`.
- If present: leave alone. The ADR log is append-only and user-owned post-creation.

### `/code/.gitignore` — create-if-missing

- If absent: write `code-gitignore-template` verbatim to `/code/.gitignore`.
- If present: leave alone.

## Step 6 — Scaffold the code wiki

Apply the same per-artifact logic that ideation `init` Step 6 uses. Each block is independent — re-runs evaluate every block.

### `/code/wiki/` and `/code/wiki/pages/` — create if missing

`mkdir -p /code/wiki/pages`.

### `/code/wiki/memory-protocol.md` — create-if-missing, additive-merge if present

This block must run on every invocation. The reference is the canonical protocol at `${PLUGIN_ROOT}/skills/init/references/memory-protocol.md` (the same file ideation `init` uses).

- If absent: write the contents of the reference verbatim. Track as `created`.
- If present: additive merge. Compare section by section, add any new sections or bullets, never remove or shorten. Track as `refreshed` if anything was added, else `already present`.

### `/code/wiki/index.md` — create-if-missing

If absent, write the template below. If present, leave alone.

```markdown
# Code Wiki Index — [PROJECT_NAME]

The content catalog for this project's code wiki. Read this first when picking up code work — it points to the relevant pages. One-line summaries only; drill into individual pages for detail.

> **Current next action:** see [next.md](next.md) — the single-file session hand-off, refreshed each session.

## Architecture
_(High-level structure, module boundaries, data flow.)_

_None yet._

## Modules
_(One page per significant module/component.)_

_None yet._

## Patterns
_(Recurring conventions: error handling, logging, config, etc.)_

_None yet._

## Bugs / Incidents
_(Postmortems, gnarly bugs worth remembering.)_

_None yet._

## Decisions
_(Index only — entries link to `/code/DECISIONS.md#d-xxx`. The ADR log is the durable record.)_

_None yet._

## Open Questions
_(Unresolved code-side questions to revisit.)_

_None yet._
```

### `/code/wiki/log.md` — create-if-missing, else append

If absent, create with the template below (substitute `{{PROJECT_NAME}}` and `{{DATE}}`). If present, append a `template-sync` entry following the same pattern as ideation `init`.

```markdown
# Code Wiki Log — {{PROJECT_NAME}}

Append-only chronological record of code-side operations: promotions received, sessions, ADRs, report-back events, lints, releases. Each entry is prefixed with `## [YYYY-MM-DD] operation | title` so the log is grep-friendly:

```bash
grep "^## \[" wiki/log.md | tail -10
grep "^## \[.*promote" wiki/log.md
grep "^## \[.*adr" wiki/log.md
```

**Operations vocabulary:**
- **promote** — new brief arrived from ideation (auto-logged by `promote-to-code`)
- **implement** — a promoted brief was implemented (manual entry at session end)
- **report-back** — finding pushed to ideation (auto-logged by `report-back`)
- **session** — generic code-side work session (manual entry)
- **lint** — code-wiki health check
- **adr** — new D-xxx recorded outside a promotion (refactor decisions, etc.)
- **close-brief** — brief closed; annotated tag created
- **release** — version tag created

**Rules:** Append only. Never edit or delete past entries. Each entry is a few bullets, not prose.

---

## [{{DATE}}] init-code | Code workspace scaffolded
- /code/ created (or detected existing)
- git: initialized (or detected existing repo, skipped)
- /code/CLAUDE.md, README.md, DECISIONS.md, .gitignore: scaffolded
- /code/wiki/ scaffolded (memory-protocol.md, index.md, log.md, next.md)
- Status: Ready for first promotion (`/yuval-core:promote-to-code CB-XXX`)
```

The re-run `template-sync` entry (when log already exists):

```markdown
## [{{DATE}}] template-sync | code wiki refreshed
- /code/CLAUDE.md updated with latest bundled template (skills/init-code/references/)
- Additive merge applied — no content removed
- /code/wiki/memory-protocol.md: [created | refreshed | already present]
- next.md: [created (back-filled) | already present]
- Status: Code workspace refreshed
```

### `/code/wiki/next.md` — create-if-missing

If absent, write:

```markdown
# Next — {{PROJECT_NAME}} (code)

_Last updated: {{DATE}}_

## Next action

Run `/yuval-core:promote-to-code CB-XXX` to bring an open brief from ideation into this code workspace, or pick up an in-flight brief.

## Why

Code workspace just scaffolded. No brief in flight yet on the code side.

## Open threads

_None yet._

## Recent context

- Last session: init-code — code workspace scaffolded.
```

If present, leave alone.

## Step 7 — First commit (first-run only)

If MODE is first-run, make a single commit:

```bash
cd /code
git add .
git commit -m "init: scaffold code workspace"
```

If MODE is re-run-existing-repo or re-run-no-repo, do **not** commit. Re-runs leave commit decisions to the user — show the diff via `git status` + `git diff` and let the user invoke "commit and push" per the workbench commit workflow.

## Step 8 — Confirm

**First run:**

```
Code workspace initialized: [PROJECT_NAME]
/code/ created with git repo (branch: main)
/code/CLAUDE.md, README.md, DECISIONS.md, .gitignore: scaffolded
/code/wiki/ scaffolded (memory-protocol.md + index.md + log.md + next.md)
Initial commit: "init: scaffold code workspace"
Ready — author a brief with /yuval-core:write-brief, then promote with /yuval-core:promote-to-code.
```

**Re-run-existing-repo:**

```
Code workspace refreshed: [PROJECT_NAME]
Existing /code/ git repo detected — git state untouched.
/code/CLAUDE.md: [refreshed (additive) | already current]
/code/README.md: [created | already present]
/code/DECISIONS.md: [created | already present]
/code/.gitignore: [created | already present]
/code/wiki/memory-protocol.md: [created | refreshed | already present]
/code/wiki/next.md: [created (back-filled) | already present]
Wiki log entry appended.
No commits made — review diff and commit when ready.
```

**Re-run-no-repo (rare):**

Same as re-run-existing-repo, but include `git: initialized in existing /code/ folder (branch: main)` and surface that no commit was made — the user reviews the new repo state and commits.
