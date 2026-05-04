# Command — `/yuval-core:promote-to-code`

Handshake command (ideation → code). Promotes an `open` brief in `/output/briefs/CB-XXX-*.md` into the code workspace by:

1. Adding a new ADR to `/code/DECISIONS.md`.
2. Freezing the brief in place (status `open` → `promoted`, status-log entry appended).
3. Appending matching `promote` log entries to `/wiki/log.md` (ideation) and `/code/wiki/log.md` (code).
4. **(v0.11.0+, D-030):** Closing any backlog items the brief marked for closure, and archiving any ideation files the brief consumed.

This is the only command that mutates state across the ideation/code boundary outbound (the other handshake is `report-back`, which goes the other way). Per D-024, both handshakes are the only sanctioned cross-mode edges.

## Refusal cases (checked before any mutation)

The command runs all refusal checks before touching any file. If any check fails, abort cleanly with a clear message.

| # | Refusal | Resolution path surfaced to user |
|---|---|---|
| 1 | Argument missing/malformed | author a brief first via `write-brief` |
| 2 | Argument is not a brief (points at a spec/report/etc.) | only briefs in `/output/briefs/` are promotion-eligible (D-023) |
| 3 | Brief file not found | list existing briefs |
| 4 | `/code/` does not exist | run `init-code` first |
| 5 | Brief status is not `open` (already `promoted` or `closed`) | author a successor brief that supersedes |
| 6 | Brief lacks `## Goal`, `## Scope`, or `## Acceptance` | edit the brief to complete it |
| 7 | `/code/SPIRIT.md` is missing entirely (D-028 promotion gate) | run `write-brief` (which authors SPIRIT.md on first invocation) or `refresh-spirit` directly |

## Steps (after refusal checks pass)

1. **Compute next D-xxx** by scanning `/code/DECISIONS.md`.
2. **Append the ADR** to `/code/DECISIONS.md` using the standard template (Decision / Reasoning / Brief reference / Rejected alternatives).
3. **Update the brief in place** — status field `open` → `promoted`, status log entry `- {{DATE}}: promoted as D-{{DXXX}}`. Three changes only; nothing else.
4. **Append `promote` entry to `/code/wiki/log.md`** with brief path + ADR + status `ready for implementation`.
5. **Append `promote` entry to `/wiki/log.md`** (ideation side) with the same info from ideation's perspective.
6. **(v0.11.0, Step 5.5)** Close marked backlog items.
7. **(v0.11.0, Step 5.6)** Archive consumed ideation files.
8. **Confirm** with file list + closure/archive counts.

## What changed in v0.11.0 (CB-006, D-030)

The command became **wiki-aware** for the first time on its outbound side. Two new steps insert between the existing log appends and the confirmation:

### Step 5.5 — Close marked backlog items

Parses the brief for `<!-- backlog-closes: <slug1>,<slug2> -->`. If present:

- For each comma-separated slug, find the matching `- [ ] **<slug>** — <context>` line under `## Active` in `/wiki/backlog.md`.
- Strike the line and append closure annotation: `- ~~**<slug>**~~ — <context> — closed by [{{CB_ID}}]({{BRIEF_RELPATH}}) on {{DATE}}`.
- Move the rewritten line out of `## Active`, prepend to `## Closed` (reverse-chronological).
- **No deletions** — closed items stay as the audit trail.
- **Slug not found in `## Active`** → log a one-line note in the confirmation, continue. No crash.

If the marker is absent, the entire step is a silent no-op.

### Step 5.6 — Archive consumed ideation files

Parses the brief for `<!-- ideation-promotes: <slug-a>,<slug-b> -->`. If present:

- For each slug, look for `/wiki/ideation/idea-<slug>.md`.
- If found, move it to `/wiki/ideation/archive/idea-<slug>.md` (using `mv`; the workbench's ideation `/wiki/` is unversioned per D-004).
- Append a footer to the moved file: `_Promoted to [{{CB_ID}}](../../../output/briefs/CB-NNN-<slug>.md) on {{DATE}}._`
- If the archive dir doesn't exist, create it (`mkdir -p`).
- **Slug not found** → log a one-line note in the confirmation, continue. No crash.

If the marker is absent, silent no-op.

### Why markers, not promotion-time matching

The user-confirmed match between brief and backlog item happens at **brief-write time** (in `write-brief`'s Step 1.8). By the time the brief reaches promotion, the slugs are already user-approved and frozen in the brief. Promotion just executes the contract.

Two reasons this is the right shape:

1. **Synthesis stays where context is freshest.** The user is actively authoring the brief in `write-brief`; backlog items are visible in the same conversation. By promotion, the conversation may be days later — re-asking the user to match items would impose load and risk worse matches.
2. **Promotion stays mechanical and auditable.** The command never makes a judgment call. Either a marker is present (act on it) or absent (no-op). Re-promotion attempts would produce identical results — except they're prevented entirely by Refusal #5.

## Skill-wiki contract (declared in the command file's header comment)

- **Reads:** `/output/briefs/CB-XXX-*.md` (the brief), `/code/DECISIONS.md`, `/code/SPIRIT.md` (gate check), `/wiki/backlog.md` (only with `backlog-closes` marker), `/wiki/ideation/*.md` (only with `ideation-promotes` marker).
- **Writes:** `/code/DECISIONS.md` (append ADR), the brief (status field + status log only), `/code/wiki/log.md` (append), `/wiki/log.md` (append), `/wiki/backlog.md` (close marked items), `/wiki/ideation/archive/*.md` (move marked files).
- **Sparse-wiki fallback:** marker absent → side-effect skipped silently. Brief is still successfully promoted. Brief without markers behaves identically to the pre-v0.11.0 contract.

## Discipline preserved

- D-023 still holds (briefs are still the only promotion-eligible artifact).
- D-024 still holds (the command remains one of two sanctioned cross-mode edges).
- D-028 SPIRIT.md gate is unchanged.
- The brief mutation is still exactly three changes (status field + status-log entry + save); markers are author-time additions to the brief, not promotion-time additions.
- Atomicity rule preserved: read every required file at the start, bail before any write if anything is missing.

## References

- Bundled command: [`plugins/yuval-core/commands/promote-to-code.md`](../../plugins/yuval-core/commands/promote-to-code.md)
- ADRs: [D-023](../../DECISIONS.md#d-023), [D-024](../../DECISIONS.md#d-024), [D-028](../../DECISIONS.md#d-028) (SPIRIT.md gate), [D-029](../../DECISIONS.md#d-029) (workshop layer), [D-030](../../DECISIONS.md#d-030) (skill-wiki contract — this command's wiki side-effects).
- Concept: [`concept-skill-wiki-contract.md`](concept-skill-wiki-contract.md), [`concept-wiki-workshop.md`](concept-wiki-workshop.md).
- Brief: [`CB-006`](../../../output/briefs/CB-006-wiki-aware-skills.md).
- Sibling skill: [`skill-write-brief`](skill-write-brief.md) — author of the markers this command consumes.
