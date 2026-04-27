---
name: init
description: >
  Initialize a new project workspace with standard folder structure, wiki scaffolding, and
  configuration. Use when the user says "init", "initialize", "set up this project", "create
  project structure", or asks to bootstrap a new workspace folder. Triggers on any request
  to start a fresh project with the standard /input, /in-progress, /output, /wiki layout.
  Also triggers on re-runs: if the user says "refresh instructions", "update CLAUDE.md",
  "sync template", or runs init on an already-initialized project.
---

# Project Init

When triggered, execute all steps autonomously with no clarifying questions. The user's selected folder is the project root.

This skill supports two modes:

- **First run** — Full initialization: create folders, write CLAUDE.md from template, scaffold the wiki, start the log.
- **Re-run** — Additive update: compare the existing CLAUDE.md against the latest template and merge in any new content. Never remove or shrink existing content — only add.

---

## Step 1 — Create Subfolders

Create these subfolders in the project root if they don't already exist:

```
/input/
/in-progress/
/output/
/output/briefs/
/wiki/
/wiki/pages/
```

Use `mkdir -p` via Bash to create them. `/wiki/pages/` is where individual wiki pages live; `/wiki/` itself holds `index.md` and `log.md`. `/output/briefs/` is the home for code briefs (`CB-XXX`) — the only artifact promoted into `/code/` via `/yuval-core:promote-to-code`. The folder is created on every project (always-on) so a project can author its first brief without a separate scaffolding step, even if the project never grows a `/code/` workspace.

## Step 2 — Detect Project Info

Determine these values from the project folder:

- **PROJECT_NAME**: The name of the current project folder (basename of the mounted path)
- **DATE**: Today's date in YYYY-MM-DD format
- **PROJECT_TYPE**: Infer from the folder name using this reference table, or default to `TBD`

| Type | Keywords to look for |
|---|---|
| Product/Strategy | product, strategy, pricing, roadmap, licensing, eol |
| Technical Research | mcp, agent, tool, architecture, research, technical |
| Content Creation | linkedin, post, deck, presentation, content, blog |
| Customer/Market Analysis | customer, market, segment, territory, partner, analysis |
| Brainstorm/Exploration | brainstorm, explore, ideation, thinking, draft |

- **PROJECT_GOAL**: Infer a one-line goal from the folder name, or default to `TBD — update before first task`

## Step 3 — Read the Template, Persona, and Memory Protocol

Three files are bundled with this skill in the `references/` folder. Version control and updates happen through the plugin's git repo — edits are made there, pushed, and propagated via plugin updates.

**Template path:** `${SKILL_DIR}/references/claude-template.md`
**Persona path:** `${SKILL_DIR}/references/persona.md`
**Memory protocol path:** `${SKILL_DIR}/references/memory-protocol.md`

Read all three files using the Read tool.

**If any file is missing:** This indicates a broken plugin installation. Warn the user and stop.

## Step 4 — Merge Persona Into Template

If `persona.md` was successfully read in Step 3, replace the persona block inside the template before doing anything else with it.

**How:** In the template content, locate the block between `<!-- PERSONA:START -->` and `<!-- PERSONA:END -->` (inclusive of the markers). Replace everything between those two markers with the content of `persona.md` (excluding the top-level `# Persona — …` heading and any HTML comment preamble at the top of `persona.md`). Keep the `<!-- PERSONA:START -->` and `<!-- PERSONA:END -->` markers in place — they're needed for future `/refresh-persona`.

If no persona markers are found in the template, skip this merge step and log a warning in the final confirmation. This means an older template is in use.

## Step 5 — Write or Update CLAUDE.md

Check whether `CLAUDE.md` already exists in the project root.

### If CLAUDE.md does NOT exist (first run)

Perform placeholder substitution on the (persona-merged) template:

- `{{PROJECT_NAME}}` → detected project name
- `{{DATE}}` → today's date
- `{{PROJECT_TYPE}}` → inferred or TBD
- `{{PROJECT_GOAL}}` → inferred or TBD

Write the result to the project root as `CLAUDE.md`.

### If CLAUDE.md ALREADY exists (re-run / update)

Additive merge. Goal: bring the existing CLAUDE.md up to date with the latest template without losing anything the user or previous sessions have added.

**How the merge works:**

1. Read the existing `CLAUDE.md` from the project root.
2. Read and substitute placeholders in the (persona-merged) template — but preserve the original `Created` date from the existing file; don't overwrite it with today's date.
3. Compare the two documents **section by section** (sections are delimited by `## ` headings).
4. Apply these rules:
   - **New section in template that doesn't exist in CLAUDE.md** → Append it in the same position it appears in the template (or at the end if positioning is ambiguous).
   - **Section exists in both** → Compare content line by line. If the template section has new bullet points, list items, table rows, checklist items, or paragraphs that aren't present in the existing section, add them to the end of that section. Never remove existing lines.
   - **Section exists in CLAUDE.md but NOT in template** → Keep it as-is. The user added it intentionally.
   - **Subsections (### level)** → Apply the same additive logic recursively.
5. Write the merged result back to `CLAUDE.md`.

**Important principles:**
- This merge is strictly additive. It must never delete, shorten, or overwrite existing content.
- If a line in the existing file was modified by the user (e.g., they customized a rule), keep the user's version. Only add lines that are entirely new in the template.
- When in doubt, keep existing content and append new content below it.
- **Exception for persona block**: on re-run, do NOT touch content between `<!-- PERSONA:START -->` and `<!-- PERSONA:END -->`. Persona refresh is a deliberate act reserved for a future `/refresh-persona` command — init should not silently overwrite a user's in-project persona.

## Step 5.5 — Prune Deprecated Sections (re-run only)

On first run, skip this step. On re-run:

1. Read `skills/init/references/deprecated-sections.md`.
2. For each section heading listed under `## Currently deprecated`, check whether the existing `CLAUDE.md` contains that heading.
3. If present, remove the `## Heading` line and all lines up to (but not including) the next `## ` heading or the end of file.
4. Track the list of sections actually removed — report it in the final confirmation output (Step 7).

This prune runs *after* the additive merge in Step 5 has produced the merged CLAUDE.md but *before* writing to disk — prune, then write. (Alternatively: run before the merge, apply merge on top; both produce the same result since the template no longer contains these sections.)

## Step 6 — Scaffold the Wiki

Handle each wiki artifact independently. Each block below specifies its own create/skip/append behavior — **do not treat this step as "first run only."** On a re-run, every block must still be evaluated because older projects may be missing artifacts introduced by later versions of the skill (e.g. `next.md`, `memory-protocol.md`).

### `/wiki/memory-protocol.md` — create-if-missing, additive-merge if present

**This block must run on every invocation.** It is the propagation path for memory-protocol updates: edits to `${SKILL_DIR}/references/memory-protocol.md` reach existing projects through `init` re-runs.

- **If `/wiki/memory-protocol.md` does not exist:** write the contents of `${SKILL_DIR}/references/memory-protocol.md` verbatim to it. Track this as `created`.
- **If `/wiki/memory-protocol.md` already exists:** apply the same additive-merge logic used for CLAUDE.md (Step 5 re-run branch) — compare section by section, add any new sections or bullets from the bundled reference, never remove or shorten existing content. Track as `refreshed` if anything was added, otherwise `already present`.

The `memory-protocol.md` reference file is the canonical memory protocol. It is owned by this plugin. Projects receive updates by re-running `init` after the plugin updates.

**Track the resulting state** (`created` / `refreshed` / `already present`) and pass it through to the `log.md` template-sync entry and the final confirmation in Step 7.

### `/wiki/index.md` — create-if-missing

If `/wiki/index.md` does not exist, create it with the template below. If it already exists, leave it alone — the wiki maintainer owns it.

```markdown
# Wiki Index — [PROJECT_NAME]

The content catalog for this project's wiki. Read this first when answering any question — it points to the relevant pages. One-line summaries only; drill into individual pages for detail.

> **Current next action:** see [next.md](next.md) — the single-file session hand-off, refreshed each session.

## Briefs
_(Code briefs (`CB-XXX`) authored in `/output/briefs/`. Each entry shows current lifecycle status — `(open)`, `(promoted: D-yyy)`, or `(closed: brief/CB-XXX)`. Empty until the project authors its first brief.)_

_None yet._

## Sources
_(Raw sources that have been ingested from /input/. Each entry links to a summary page in /wiki/pages/.)_

_None yet._

## Entities
_(People, companies, products, systems that appear across multiple sources.)_

_None yet._

## Concepts
_(Ideas, frameworks, recurring themes. One page per concept.)_

_None yet._

## Decisions
_(Decisions made during the project, with rationale. One page per decision.)_

_None yet._

## Open Questions
_(Unresolved questions to revisit as more sources arrive.)_

_None yet._
```

### `/wiki/log.md` — create-if-missing, else append

If `/wiki/log.md` does not exist, create it with the init entry:

```markdown
# Wiki Log — [PROJECT_NAME]

Append-only chronological record of ingests, queries, lint passes, and sessions. Each entry is prefixed with `## [YYYY-MM-DD] operation | title` for grep-friendly scanning:

    grep "^## \[" wiki/log.md | tail -10

---

## [DATE] init | Project initialized
- Folders created: /input /in-progress /output /output/briefs /wiki /wiki/pages
- CLAUDE.md configured from template
- Persona injected from persona.md
- Wiki scaffolded (memory-protocol.md, index.md, log.md, next.md)
- Status: Ready for first task
```

If `/wiki/log.md` already exists, append a `template-sync` entry. Fill in the `next.md:` line using the flag tracked in the `/wiki/next.md` block below — use `created (back-filled for pre-continuity project)` if the file was created this run, otherwise `already present`. Fill in the `memory-protocol.md:` line using the flag tracked in the `/wiki/memory-protocol.md` block above — one of `created`, `refreshed`, or `already present`. Fill in `Deprecated sections removed:` with the comma-separated list tracked in Step 5.5, or `none` if nothing was pruned.

```markdown
## [DATE] template-sync | CLAUDE.md refreshed
- CLAUDE.md updated with latest bundled template (skills/init/references/)
- Additive merge applied — no content removed outside the sanctioned deprecated-sections channel
- Persona block preserved (refresh persona separately)
- Deprecated sections removed: [list or "none"]
- next.md: [created (back-filled for pre-continuity project) | already present]
- memory-protocol.md: [created | refreshed | already present]
- Status: Instructions refreshed
```

### `/wiki/next.md` — create-if-missing (applies on re-run too)

**This block must run on every invocation, including re-runs.** Older projects that predate the session continuity contract won't have `next.md`, and the re-run must back-fill it — otherwise the contract defined in CLAUDE.md has nothing to read.

If `/wiki/next.md` does not exist, create it with the empty hand-off below. If it already exists, leave it alone.

**Track whether `next.md` was created this run.** Pass this flag through to the `log.md` template-sync entry above and to the final confirmation in Step 7.

```markdown
# Next — [PROJECT_NAME]

_Last updated: [DATE]_

## Next action
Drop source material into `/input/` and describe your first task.

## Why
Project was just initialized — no work in flight yet.

## Open threads
_None yet._

## Recent context
- Last session: init — project scaffolded.
```

The session continuity contract itself lives in CLAUDE.md: read `next.md` at session start, overwrite and print it at session end.

### `/input/README.md` — create-if-missing

If `/input/README.md` does not exist, create it with the template below. If it already exists, leave it alone.

```markdown
# Input — [PROJECT_NAME]

Drop all source materials here before starting a task.
Claude scans this folder at the start of every session.

To ingest a file into the wiki, drop it here and say "ingest [filename]" or "ingest everything new."
```

## Step 7 — Confirm

**First run** — respond with:

```
Project initialized: [Project Name]
Folders: /input /in-progress /output /output/briefs /wiki
CLAUDE.md configured (template: skills/init/references/claude-template.md)
Persona injected (source: skills/init/references/persona.md)
Wiki scaffolded (memory-protocol.md + index.md + log.md + next.md)
next.md: created
memory-protocol.md: created
Ready — drop files into /input and describe your first task. To scope code work, run `/yuval-core:write-brief`.
```

**Re-run** — respond with the form below. Fill the `next.md:` line with `created (back-filled for pre-continuity project)` if the file was created this run, otherwise `already present`. Fill the `memory-protocol.md:` line with one of `created`, `refreshed`, or `already present` based on the flag tracked in Step 6. Fill `Deprecated sections removed` with the comma-separated list from Step 5.5, or `none` if nothing was pruned. No bracket-OR notation in the output.

```
Project updated: [Project Name]
CLAUDE.md refreshed from skills/init/references/claude-template.md
New sections/items added — no existing content removed outside the deprecated-sections channel
Persona block preserved (use /refresh-persona to update persona separately)
Deprecated sections removed: [list or "none"]
Wiki log entry appended
next.md: [created (back-filled for pre-continuity project) | already present]
memory-protocol.md: [created | refreshed | already present]
```
