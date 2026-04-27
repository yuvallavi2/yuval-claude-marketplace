---
name: write-brief
description: >
  Author a code brief (CB-XXX) interactively. Use when the user says "write a brief",
  "draft a brief", "new brief", "scope this for code", "author CB", or otherwise asks
  to capture a scoped commitment for code-mode implementation. Walks goal → scope →
  out-of-scope → references → acceptance, computes the next CB-XXX number, and writes
  to `/output/briefs/CB-XXX-<slug>.md` with status `open`.
---

# Write Brief

When triggered, walk the user through authoring a code brief. The result is a single file in `/output/briefs/CB-XXX-<slug>.md` ready for promotion via `/yuval-core:promote-to-code`.

A brief is the **contract artifact** between ideation and code. It is the only artifact that becomes promotion-eligible. Specs, design docs, and reports stay in `/output/` as reference material — briefs link to them but are not the same thing.

---

## Step 1 — Verify the workspace

1. Confirm `/output/briefs/` exists. If not, create it (`mkdir -p /output/briefs`). Note: the ideation `init` skill (v0.6.0+) creates this folder by default; absence implies an older project.
2. Read `${SKILL_DIR}/references/brief-template.md`. If missing, abort — broken plugin install.

## Step 2 — Compute the next CB-XXX number

List existing files in `/output/briefs/` matching the pattern `CB-NNN-*.md`. Take the highest `NNN` and increment by one. Pad to three digits (`CB-001`, `CB-014`, `CB-127`). If the folder is empty, start at `CB-001`.

## Step 3 — Walk the user through authoring

Ask **one question at a time** (per the workbench's user-collaboration norms). Do not stack questions.

The order:

### 3.1 Title

> "What's the brief about? Give me a short title (5–10 words)."

Use the title to compute the slug:
- Lowercase the title.
- Replace non-alphanumeric characters with `-`.
- Collapse runs of `-` to a single `-`.
- Trim leading/trailing `-`.

Example: `"Bootstrap code workspace extension"` → `bootstrap-code-workspace-extension`.

### 3.2 Goal

> "One sentence — what coding phase is this scoping?"

This is the brief's `## Goal` section. Keep it tight. If the user gives prose, distill to one sentence and confirm.

### 3.3 Scope (to-do)

> "What needs to happen? List the concrete deliverables — I'll capture them as checkboxes."

Capture as a `## Scope (to-do)` checklist. Each item: `- [ ] <verb-phrase>`. If the user gives bulky paragraphs, propose checklist items and confirm before writing.

### 3.4 Out of scope

> "What is this brief explicitly *not* covering? — anything that's tempting to bundle but should stay out."

This is the **most important** section of a brief — it prevents scope creep mid-implementation. Push for at least one item, even if the user shrugs. Common candidates: testing harness, future enhancements, migration of unrelated projects, performance work.

### 3.5 References

> "What docs, specs, or wiki pages does this brief depend on?"

Capture as relative-path links from the brief's location (`/output/briefs/`):
- Specs in `/output/`: `[`/output/foo.md`](../foo.md)`
- Wiki pages: `[`/wiki/pages/foo.md`](../../wiki/pages/foo.md)`
- Predecessor briefs: `[CB-XXX](CB-XXX-<slug>.md)` if a prior brief is being superseded or extended.

### 3.6 Acceptance

> "How will we know the brief is done? — one or two lines, or a numbered checklist if there are multiple criteria."

This is the closure contract. Be specific. Avoid vibes ("the feature works"). Prefer measurable criteria ("scope items 1–6 shipped under `/code/plugins/...`; D-xxx recorded; tests in spec §10 pass").

## Step 4 — Write the brief

Substitute the captured values into `references/brief-template.md`:

- `{{CB_NUMBER}}` → `CB-001`, `CB-002`, etc.
- `{{TITLE}}` → the user's title
- `{{DATE}}` → today's date in `YYYY-MM-DD`
- `{{GOAL}}` → the one-sentence goal
- `{{SCOPE_ITEMS}}` → the markdown checklist
- `{{OUT_OF_SCOPE_ITEMS}}` → the markdown bullet list
- `{{REFERENCES}}` → the markdown link list
- `{{ACCEPTANCE}}` → the acceptance criteria

Write to `/output/briefs/CB-XXX-<slug>.md`. Status field is `open`. ADR field is `TBD`.

## Step 5 — Confirm

Respond with:

```
Brief authored: CB-XXX — [Title]
Path: /output/briefs/CB-XXX-<slug>.md
Status: open
Next: review/edit until you're satisfied, then run `/yuval-core:promote-to-code CB-XXX` to freeze it and create the ADR.
```

Print the **path** (clickable for the user) and a one-line summary. Do not paste the entire brief contents back — the user just authored it.

---

## Notes

- **Brief vs spec:** A brief is a *commitment*. A spec is a *description*. If the user is describing how something works, that's a spec — direct it to `/output/` (or `/in-progress/` if it's a draft), not `/output/briefs/`.
- **Edit before promotion:** Briefs stay `open` and editable until `promote-to-code` runs. Encourage the user to refine before promoting — promotion freezes the brief.
- **Reference, don't inline:** A brief should link to specs by relative path, not inline the spec content. Keeps briefs short and maintainable.
- **No automatic logging:** Authoring a brief does not append to `/wiki/log.md`. Only promotion does. The brief file's existence in `/output/briefs/` is the record. (Optional: if the user explicitly asks, add a `brief` log entry — but it's not the default.)
