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

<!--
Skill-wiki contract (D-030):
- Declared READS:  /code/SPIRIT.md, /wiki/goals.md, /wiki/backlog.md, /wiki/ideation/*.md (when the user points at one in References)
- Declared WRITES: /output/briefs/CB-XXX-<slug>.md (the brief itself, including <!-- backlog-closes: ... --> and <!-- ideation-promotes: ... --> markers stashed for promote-to-code to consume)
- This skill never writes to /wiki/* directly. Wiki side-effects from a brief happen at promotion, via the markers below.
- Sparse-wiki fallback: every wiki read is best-effort. Missing or malformed file → skip the corresponding step silently and continue.
-->

---

## Step 1 — Verify the workspace

1. Confirm `/output/briefs/` exists. If not, create it (`mkdir -p /output/briefs`). Note: the ideation `init` skill (v0.6.0+) creates this folder by default; absence implies an older project.
2. Read `${SKILL_DIR}/references/brief-template.md`. If missing, abort — broken plugin install.
3. Read `${SKILL_DIR}/references/spirit-prompts.md`. If missing, abort — broken plugin install.

## Step 1.5 — SPIRIT.md gate (D-028)

Before authoring the brief, the project's spirit must be set. SPIRIT.md is the **project-level** single source of truth for atmosphere; it is authored once on the first brief and refreshed via `/yuval-core:refresh-spirit` if it ever needs to evolve.

Check whether `/code/SPIRIT.md` exists at the workspace root.

### 1.5.a — Absent path (first brief in this project)

`/code/SPIRIT.md` does not exist. Walk the 8 spirit prompts inline, **one at a time**, before any of the brief authoring questions below. The full prompts and example answers are bundled at `${SKILL_DIR}/references/spirit-prompts.md` — read it now and use the eight question lines verbatim.

The 8 prompts, in order:

1. Reference Scenario
2. Interaction Shape
3. Voice / Tone samples
4. Anti-Shape
5. Silent State
6. Tactile Contract
7. Default Posture
8. The One Moment

Rules for the walk:
- Ask one prompt per turn. Do not stack prompts.
- `n/a` is a valid answer for any prompt. Don't push back on `n/a` — the prompts are universal but not every project has user-facing surface. Pure-infrastructure projects answer `n/a` to all 8.
- If all 8 answers are `n/a`, write `/code/SPIRIT.md` as a single line:
  ```
  n/a — pure infrastructure project. No user surface, no atmosphere to transmit.
  ```
- Otherwise, write `/code/SPIRIT.md` with one `## <prompt name>` heading per non-`n/a` answer, with the user's answer underneath. Skip prompts answered `n/a`. Top of file:
  ```markdown
  # SPIRIT — <project name>

  _Authored: YYYY-MM-DD via `/yuval-core:write-brief` on the first brief. Refresh via `/yuval-core:refresh-spirit`._

  <body: one section per non-n/a prompt>
  ```
- After the file is written, confirm to the user: *"`/code/SPIRIT.md` written. Continuing with the brief — its `## Spirit & Texture` section will read `Inherits /code/SPIRIT.md.`"* Then continue to Step 2.

### 1.5.b — Present path (subsequent briefs in this project)

`/code/SPIRIT.md` exists. Ask **one** question:

> *"Any spirit adjustment for this brief? (default: no)"*

- **Default / "no" / silence** → set the brief's `## Spirit & Texture` section to `Inherits /code/SPIRIT.md.` (the template default; no special handling needed).
- **"yes" + an override paragraph** → capture the one-paragraph override. Set the brief's `## Spirit & Texture` section to:
  ```markdown
  Inherits [/code/SPIRIT.md](../../code/SPIRIT.md), except for this brief:

  <one-paragraph override from the user>
  ```

If the user's override balloons past one paragraph, gently note: *"That's substantial — sounds like SPIRIT.md itself should evolve. I'll capture this short for the brief; consider running `/yuval-core:refresh-spirit` after."* Don't block on it; capture what fits and continue.

Continue to Step 2.

### 1.5.c — Workspace lacks `/code/`

If `/code/` does not exist at all (the project hasn't been promoted to code yet), skip the gate — the brief is being authored before `init-code`. Note to the user: *"`/code/` does not exist yet, so SPIRIT.md authoring is deferred. After you run `/yuval-core:init-code`, the next `/yuval-core:write-brief` invocation will walk the spirit prompts before that brief is authored."* Then continue to Step 2 with the brief's `## Spirit & Texture` section omitted (the brief will be edited or re-authored once `/code/` exists).

## Step 1.7 — Read goals (D-030)

Read `/wiki/goals.md` if it exists.

- **File missing or `## Active` section absent / empty:** silently skip — no surface to the user, no brief mutation. Continue to Step 1.8.
- **File present with active goals:** parse the `## Active` section into a list of goal titles (one per `- [ ]` checkbox line). Surface them to the user in one turn:

  > *"Current active goals:*
  > *<numbered list of titles>*
  >
  > *Is this brief part of one of them, or a new direction?"*

  Capture the user's answer (free text, often a number or short phrase). Stash the answer for Step 4 — the brief's References section will gain a `> Goal alignment: <answer>` blockquote line.

Do **not** gate the brief on goal alignment. The point is surfacing the connection, not enforcing it. A brief that's a new direction is still a valid brief.

## Step 1.8 — Read backlog (D-030)

Read `/wiki/backlog.md` if it exists.

- **File missing or `## Active` section absent / empty:** silently skip. Continue to Step 2.
- **File present with active items:** parse the `## Active` section into a list (one per `- [ ]` checkbox line; the slug is the bolded `**<slug>**` token). After Step 3.1 (Title) has run — so you have a working title — use Claude judgment to identify which backlog items semantically match the brief's emerging title and scope. **No fuzzy-string matching, no tag matching — read the items and judge.**
  - If zero matches, silently continue.
  - If one or more matches, ask the user (one turn):

    > *"This brief looks like it could close backlog items:*
    > *<numbered list of matching items, with their slugs>*
    >
    > *Close any of them when this brief is promoted? (numbers, all, none)"*

    Parse the user's answer into a list of slugs. Stash the slug list for Step 4 — the brief will gain a `<!-- backlog-closes: <slug1>,<slug2>,... -->` HTML comment that `promote-to-code` will consume.

This step requires Step 3.1 to have run. Defer the actual ask until after the title is captured. (The backlog **read** can happen now, but the **ask** comes after Step 3.1.)

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

> "What docs, specs, or wiki pages does this brief depend on? You can also point at recent ideation files (`/wiki/ideation/idea-<slug>.md`) — if so, the file will be archived to `/wiki/ideation/archive/` at promotion."

Capture as relative-path links from the brief's location (`/output/briefs/`):
- Specs in `/output/`: `[`/output/foo.md`](../foo.md)`
- Wiki pages: `[`/wiki/pages/foo.md`](../../wiki/pages/foo.md)`
- Predecessor briefs: `[CB-XXX](CB-XXX-<slug>.md)` if a prior brief is being superseded or extended.
- Ideation files: `[`/wiki/ideation/idea-<slug>.md`](../../wiki/ideation/idea-<slug>.md)` — also stash the slug so Step 4 can append a `<!-- ideation-promotes: <slug> -->` marker.

If the user names one or more ideation files, also append a `> Goal alignment: <answer>` blockquote (from Step 1.7) and any goal-related context to the References section. Multiple ideation slugs become a comma-separated list in the marker: `<!-- ideation-promotes: <slug1>,<slug2> -->`.

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
- `{{SPIRIT_AND_TEXTURE}}` → the contents of the brief's `## Spirit & Texture` section, computed in Step 1.5:
  - **Default (no per-brief adjustment, or Step 1.5.a just authored SPIRIT.md):** `Inherits [/code/SPIRIT.md](../../code/SPIRIT.md).`
  - **With override:** `Inherits [/code/SPIRIT.md](../../code/SPIRIT.md), except for this brief:\n\n<override paragraph>`
  - **No `/code/` yet (Step 1.5.c):** omit the section header and body — leave `{{SPIRIT_AND_TEXTURE}}` as an empty string and remove the `## Spirit & Texture` header from the rendered brief.

Write to `/output/briefs/CB-XXX-<slug>.md`. Status field is `open`. ADR field is `TBD`.

### Marker comments (D-030)

If Step 1.8 captured backlog closures or Step 3.5 captured ideation pointers, insert HTML comment markers into the brief immediately after the status line. `promote-to-code` parses these on promotion.

```markdown
# CB-XXX — Title

_Created: YYYY-MM-DD | Status: open | ADR: TBD_
<!-- backlog-closes: <slug1>,<slug2> -->
<!-- ideation-promotes: <slug-a>,<slug-b> -->

## Goal
...
```

Either marker is omitted if its corresponding list is empty. Both markers carry comma-separated slug lists (no spaces around commas). The user-confirmed slugs in Step 1.8 / Step 3.5 are the explicit hand-off contract for the wiki side-effects that `promote-to-code` will execute. Briefs without markers cause no wiki side-effects at promotion — the existing promotion contract is unchanged.

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
- **SPIRIT.md altitude (D-028):** Spirit is project-level, set once. The first brief in a project authors SPIRIT.md inline; every subsequent brief inherits it with at most a one-paragraph adjustment. If a brief's adjustment is substantial enough to warrant more than a paragraph, that's a signal the project's SPIRIT.md itself should evolve — point the user at `/yuval-core:refresh-spirit` rather than letting the override grow.
