---
description: Promote an open brief (CB-XXX) into the code workspace — adds an ADR to /code/DECISIONS.md, freezes the brief, logs both wikis.
argument-hint: CB-XXX
---

# /yuval-core:promote-to-code

Handshake command: ideation → code. Promotes a single brief in `/output/briefs/CB-XXX-<slug>.md` into the code workspace by adding a new ADR to `/code/DECISIONS.md`, freezing the brief in place, and appending matching `promote` entries to both `/wiki/log.md` (ideation) and `/code/wiki/log.md` (code).

**Argument:** `$ARGUMENTS` should be a brief identifier in the form `CB-NNN` (e.g., `CB-007`). If empty, error and instruct the user to pass one.

---

## Refusal cases — check before any mutation

Before touching any file, validate the request. Refuse with a clear message in any of these cases:

1. **Argument missing or malformed.** Expected `CB-NNN` (three digits, optionally more). If the user passed nothing or something unrecognizable, say: *"`/yuval-core:promote-to-code` expects a brief ID like `CB-007`. Author a brief first with `/yuval-core:write-brief`, then promote it."*

2. **Argument is not a brief.** If the user passes a path or a name that points at a non-brief artifact in `/output/` (e.g., a spec, a report, a design doc), refuse: *"Only briefs in `/output/briefs/` are promotion-eligible. Other `/output/` artifacts are reference material — link to them from a brief, then promote the brief. Author one with `/yuval-core:write-brief`."* (See ADR D-023 for the rationale — link to it.)

3. **Brief file not found.** Look in `/output/briefs/` for a file matching `CB-NNN-*.md`. If no match, say: *"No brief found at `/output/briefs/CB-NNN-*.md`. Did you mean a different ID? Existing briefs:"* and list them.

4. **`/code/` does not exist.** If `/code/` is missing, say: *"This project hasn't been promoted to code yet. Run `/yuval-core:init-code` first to scaffold the code workspace, then re-run promotion."*

5. **Brief status is not `open`.** Read the brief's status field (line under the title: `_Created: ... | Status: ... | ADR: ...`). If status is `promoted` or `closed`, refuse: *"`CB-NNN` is already `<status>` (ADR `D-xxx`). Promotion is one-way — to supersede, author a successor brief that references this one as predecessor."*

6. **Brief lacks required sections.** Verify the brief has `## Goal`, `## Scope`, `## Acceptance`. If any are missing, refuse: *"`CB-NNN` is missing `<section>`. Edit the brief to complete it before promoting."*

7. **`/code/SPIRIT.md` is missing entirely.** Per D-028, project spirit must be authored at the project level before any brief can be promoted. If `/code/SPIRIT.md` does not exist, refuse: *"`/code/SPIRIT.md` does not exist. Spirit must be authored at the project level before any brief can be promoted. Run `/yuval-core:write-brief` (which will walk the spirit prompts on first invocation) or `/yuval-core:refresh-spirit` directly."* This check applies to every brief, including pure-infrastructure projects — the `n/a — pure infrastructure project. ...` form of SPIRIT.md is acceptable and passes the check, but a missing file does not.

If all checks pass, proceed with the steps below — **do not** ask the user to confirm. Promotion is the explicit confirmation; the command's contract is to execute it.

---

## Steps

### 1. Compute the next D-xxx number

Read `/code/DECISIONS.md`. Find the highest existing `## D-NNN` heading. Increment by one. Pad to three digits if existing entries are padded; otherwise no padding.

### 2. Append the ADR to `/code/DECISIONS.md`

Append (do not overwrite). The ADR uses this template — substitute the brief's title and goal:

```markdown
---

## D-{{DXXX}} — {{BRIEF_TITLE}}

**Decision:** Implement the scope captured in [`{{BRIEF_RELPATH}}`]({{BRIEF_RELPATH}}) — see brief for full to-do list, out-of-scope items, and acceptance criteria.

**Reasoning:** {{BRIEF_GOAL}}

**Brief reference:** [`{{BRIEF_RELPATH}}`]({{BRIEF_RELPATH}}) — promoted {{DATE}}, status `promoted` post-promotion. The brief is immutable post-promotion and serves as the as-of-promotion snapshot of scope.

**Rejected alternatives:** _(see brief's `## Out of scope` for what was explicitly excluded.)_
```

Where:
- `{{DXXX}}` is the computed ADR number (e.g., `D-019`, `D-027`).
- `{{BRIEF_TITLE}}` is the H1 title of the brief (the line `# CB-NNN — Title`, just the title portion).
- `{{BRIEF_RELPATH}}` is the relative path from `/code/DECISIONS.md` to the brief — typically `../output/briefs/CB-NNN-<slug>.md`.
- `{{BRIEF_GOAL}}` is the content of the brief's `## Goal` section.
- `{{DATE}}` is today's date in `YYYY-MM-DD`.

### 3. Update the brief in place

This is **the only sanctioned mutation of an `open` brief.** Apply exactly these three changes — nothing else:

1. Replace the status line:
   - From: `_Created: YYYY-MM-DD | Status: open | ADR: TBD_`
   - To: `_Created: YYYY-MM-DD | Status: promoted | ADR: D-{{DXXX}}_`
2. Append to the brief's `## Status log` section:
   - `- {{DATE}}: promoted as D-{{DXXX}}`
3. Save.

Do not edit any other section of the brief. The brief is the as-of-promotion snapshot of scope; promotion freezes it.

### 4. Append `promote` entry to `/code/wiki/log.md`

```markdown
## [{{DATE}}] promote | {{CB_ID}} → D-{{DXXX}}
- Brief: {{BRIEF_RELPATH}}
- ADR: D-{{DXXX}} in DECISIONS.md
- Status: ready for implementation
```

Where `{{CB_ID}}` is `CB-NNN` and `{{BRIEF_RELPATH}}` is the path from `/code/wiki/log.md` to the brief — typically `../../output/briefs/CB-NNN-<slug>.md`.

### 5. Append `promote` entry to `/wiki/log.md` (ideation)

```markdown
## [{{DATE}}] promote | {{CB_ID}} → /code/
- Brief: /output/briefs/CB-NNN-<slug>.md
- Promoted as: D-{{DXXX}}
- Implementation: pending
```

### 6. Confirm

Respond with the list of files touched:

```
Promoted: CB-NNN — [Brief Title]
ADR added: D-{{DXXX}} in /code/DECISIONS.md
Brief frozen: /output/briefs/CB-NNN-<slug>.md (status → promoted)
Logs updated: /wiki/log.md, /code/wiki/log.md
Next: implement the brief's scope. Close with an annotated `brief/CB-NNN` tag at the scope-completion commit.
```

---

## Notes for the implementer

- **Atomicity:** If any step fails (e.g., `/code/wiki/log.md` doesn't exist because `init-code` was never run), abort the entire operation cleanly. Do not leave the brief partially mutated. Best practice: read every required file at the start of step 1; bail before any write if anything is missing.
- **Diff review:** This command makes mutations the user will likely review later (especially the ADR text). Show a brief summary of what was added but do not require approval mid-flow — the explicit `/yuval-core:promote-to-code CB-XXX` invocation is the approval.
- **Bootstrap caveat:** This command does not exist for `CB-001` itself — `CB-001` is the brief that scopes building this command. `CB-001` was promoted by hand-authoring D-019 in `/code/DECISIONS.md` and manually editing the brief and both logs. The hand-authored steps were the validation that this command's contract is correct.
