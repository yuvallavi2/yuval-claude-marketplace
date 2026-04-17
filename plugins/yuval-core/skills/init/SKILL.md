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

Create these four subfolders in the project root if they don't already exist:

```
/input/
/in-progress/
/output/
/wiki/
/wiki/pages/
```

Use `mkdir -p` via Bash to create them. `/wiki/pages/` is where individual wiki pages live; `/wiki/` itself holds `index.md` and `log.md`.

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

## Step 3 — Read the Template and Persona

Both files live in the user's marketplace folder, not bundled with this skill. This way the user edits them once and every future project picks up the latest versions automatically.

**Template path:** `~/claude-marketplace/templates/claude-template.md`
**Persona path:** `~/claude-marketplace/templates/persona.md`

Read both files using the Read tool.

**If the template doesn't exist:** Fall back to the bundled template at `${CLAUDE_PLUGIN_ROOT}/skills/init/references/claude-template.md` and inform the user they should create `~/claude-marketplace/templates/claude-template.md` to customize future projects.

**If the persona file doesn't exist:** Proceed without replacing the persona block — the template's default persona content will be used as-is. Note this in the final confirmation so the user knows to create `~/claude-marketplace/templates/persona.md` if they want a decoupled, refreshable persona.

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

## Step 6 — Scaffold the Wiki

### `/wiki/index.md` (first run only)

If `/wiki/index.md` does not exist, create it:

```markdown
# Wiki Index — [PROJECT_NAME]

The content catalog for this project's wiki. Read this first when answering any question — it points to the relevant pages. One-line summaries only; drill into individual pages for detail.

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

If it already exists, skip.

### `/wiki/log.md` (first run only)

If `/wiki/log.md` does not exist, create it with the init entry:

```markdown
# Wiki Log — [PROJECT_NAME]

Append-only chronological record of ingests, queries, lint passes, and sessions. Each entry is prefixed with `## [YYYY-MM-DD] operation | title` for grep-friendly scanning:

    grep "^## \[" wiki/log.md | tail -10

---

## [DATE] init | Project initialized
- Folders created: /input /in-progress /output /wiki /wiki/pages
- CLAUDE.md configured from template
- Persona injected from persona.md
- Wiki scaffolded (index.md, log.md)
- Status: Ready for first task
```

### On re-run

If `/wiki/log.md` already exists, append a new entry:

```markdown
## [DATE] template-sync | CLAUDE.md refreshed
- CLAUDE.md updated with latest template from ~/claude-marketplace/templates/
- Additive merge applied — no content removed
- Persona block preserved (refresh persona separately)
- Status: Instructions refreshed
```

If `/wiki/index.md` already exists, leave it alone — it's owned by the wiki maintainer, not init.

## Step 7 — Create Input README (first run only)

If `/input/README.md` does not exist, create it:

```markdown
# Input — [PROJECT_NAME]

Drop all source materials here before starting a task.
Claude scans this folder at the start of every session.

To ingest a file into the wiki, drop it here and say "ingest [filename]" or "ingest everything new."
```

If it already exists, skip.

## Step 8 — Confirm

**First run** — respond with:

```
Project initialized: [Project Name]
Folders: /input /in-progress /output /wiki
CLAUDE.md configured (template: ~/claude-marketplace/templates/claude-template.md)
Persona injected (source: ~/claude-marketplace/templates/persona.md)
Wiki scaffolded (index.md + log.md)
Ready — drop files into /input and describe your first task.
```

If the persona file was missing, add:
```
Note: ~/claude-marketplace/templates/persona.md not found. Template default persona used.
Create this file to enable decoupled persona refresh.
```

**Re-run** — respond with:

```
Project updated: [Project Name]
CLAUDE.md refreshed from ~/claude-marketplace/templates/claude-template.md
New sections/items added — no existing content removed
Persona block preserved (use /refresh-persona to update persona separately)
Wiki log entry appended
```
