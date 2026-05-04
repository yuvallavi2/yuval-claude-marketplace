# Skill — `write-brief`

Authors a code brief (`CB-XXX`) interactively. Walks goal → scope → out-of-scope → references → acceptance, computes the next CB number, writes to `/output/briefs/CB-XXX-<slug>.md` with status `open`. The brief is the only promotion-eligible artifact (D-023).

## Current scope (v0.11.0+)

The skill walks the user through nine ordered phases:

1. **Step 1** — Verify workspace (briefs folder + bundled references).
2. **Step 1.5** — SPIRIT.md gate (D-028). Three paths: absent → walk 8 prompts and write SPIRIT.md; present → ask one adjustment question; no `/code/` yet → defer.
3. **Step 1.7** — Read goals (D-030, new in v0.11.0). Loads `/wiki/goals.md` active items, asks the user whether the brief is part of one of them. Captures the alignment as a blockquote in References. Sparse-wiki fallback: missing/empty goals.md → silently skip.
4. **Step 1.8** — Read backlog (D-030, new in v0.11.0). Loads `/wiki/backlog.md` active items. After Step 3.1 (Title) runs and a working title exists, uses Claude judgment to identify backlog items that semantically match. If matches found, asks the user which (if any) the brief should close at promotion. Stashes the user's selection as a `<!-- backlog-closes: <slug1>,<slug2> -->` HTML comment marker for `promote-to-code` to consume.
5. **Step 2** — Compute the next `CB-NNN` number.
6. **Step 3.1–3.6** — Walk title → goal → scope → out-of-scope → references → acceptance, one question at a time.
   - **Step 3.5 (References)** also accepts `/wiki/ideation/idea-<slug>.md` pointers; user-selected slugs become a `<!-- ideation-promotes: <slug-a>,<slug-b> -->` marker.
7. **Step 4** — Write the brief from the bundled template, including any HTML-comment markers from Steps 1.8 / 3.5 immediately after the status line.
8. **Step 5** — Confirm with path + next-action pointer.

## What changed in v0.11.0 (CB-006, D-030)

The skill became **wiki-aware**: it now reads three wiki surfaces and stashes promotion-time markers in the brief so `promote-to-code` can act on them later.

**New reads:**

- `/wiki/goals.md` (Step 1.7) — surface active goals, ask alignment, capture as blockquote.
- `/wiki/backlog.md` (Step 1.8) — surface matching backlog items, ask closure intent, stash marker.
- `/wiki/ideation/*.md` (Step 3.5) — accept user-named ideation pointers, stash marker.

**New writes (still narrow):**

The skill still only writes one file: the brief itself at `/output/briefs/CB-XXX-<slug>.md`. The new behavior is that the brief now carries HTML-comment markers (`<!-- backlog-closes: ... -->`, `<!-- ideation-promotes: ... -->`) when the user opted into closures or archives during the walk. **No wiki writes from `write-brief` itself** — wiki side-effects happen at promotion.

**Sparse-wiki fallback:**

Every wiki read is best-effort. If `/wiki/goals.md` is missing, Step 1.7 silently skips. If `/wiki/backlog.md` is missing or empty, Step 1.8 silently skips. The skill never crashes on missing or malformed wiki files. This matches the D-030 contract (read freely, never gate on wiki availability).

**Brief markers as the hand-off contract:**

The user confirms backlog closures and ideation archives **at brief-write time**, not at promotion time. The markers are the explicit hand-off — `promote-to-code` doesn't re-ask, doesn't re-judge, just executes what the brief declared. This keeps the synthesis (LLM-judged matches) in the authoring step where context is freshest, and keeps promotion mechanical and auditable.

## Discipline preserved

- The brief still has all six standard sections (Goal / Scope / Out-of-scope / References / Acceptance / Spirit & Texture / Status log).
- Per-question single-turn discipline preserved (one question at a time per workbench norms).
- D-016 self-containment preserved (no cross-skill references; `references/spirit-prompts.md` is still bundled inside the skill).
- D-023 still holds (briefs are still the only promotion-eligible artifact; markers don't change that — they just attach wiki side-effects to the existing promotion event).
- The `## Spirit & Texture` section default (`Inherits /code/SPIRIT.md.`) is unchanged.

## Skill-wiki contract (declared in the SKILL.md header comment)

- **Reads:** `/code/SPIRIT.md`, `/wiki/goals.md`, `/wiki/backlog.md`, `/wiki/ideation/*.md`
- **Writes:** `/output/briefs/CB-XXX-<slug>.md` (the brief, including markers)
- **Sparse-wiki fallback:** every read is best-effort; missing/malformed file → skip step silently.

## References

- Bundled SKILL: [`plugins/yuval-core/skills/write-brief/SKILL.md`](../../plugins/yuval-core/skills/write-brief/SKILL.md)
- Bundled references: [`plugins/yuval-core/skills/write-brief/references/`](../../plugins/yuval-core/skills/write-brief/references/) — `brief-template.md`, `spirit-prompts.md`.
- ADRs: [D-023](../../DECISIONS.md#d-023) (briefs are the only promotion-eligible artifact), [D-028](../../DECISIONS.md#d-028) (SPIRIT.md SoT — Step 1.5 gate), [D-029](../../DECISIONS.md#d-029) (workshop layer — defines the wiki surfaces this skill reads), [D-030](../../DECISIONS.md#d-030) (skill-wiki contract — defines the read/write narrowness).
- Concept: [`concept-skill-wiki-contract.md`](concept-skill-wiki-contract.md) — the three-rule contract.
- Brief: [`CB-006`](../../../output/briefs/CB-006-wiki-aware-skills.md) (CB-005 added the wiki surfaces; CB-006 wires this skill to consume them).
