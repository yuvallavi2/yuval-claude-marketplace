# Skill — refresh-spirit

_Origin: CB-004 / D-028 — shipped in v0.9.0 (2026-05-03)._

## What it does

Re-walks the 8 spirit prompts pre-filled with the project's current `/code/SPIRIT.md` content, shows a unified diff of old vs new, and overwrites SPIRIT.md whole-file. Mirrors the contract of `refresh-persona` (D-026/D-027): single narrow mutation, no per-section preservation, abort cleanly when the source is absent.

## File layout

```
plugins/yuval-core/skills/refresh-spirit/
├── SKILL.md
└── references/
    └── spirit-prompts.md   ← copied verbatim from skills/write-brief/references/spirit-prompts.md at ship time
```

## Why mirror refresh-persona

`refresh-spirit` and `refresh-persona` solve the same shape of problem at different altitudes:

| | refresh-persona | refresh-spirit |
|---|---|---|
| **What it refreshes** | The persona block in a project's `CLAUDE.md` | The project's `/code/SPIRIT.md` |
| **Authored by** | `init` | `write-brief` (on first brief in a project) |
| **Authoritative source** | Bundled `references/persona.md` (one persona for everyone) | The project's own SPIRIT.md (one atmosphere per project) |
| **Boundary** | Marker pair `<!-- PERSONA:START -->` / `<!-- PERSONA:END -->` | Whole file |
| **Abort condition** | Marker pair missing or CLAUDE.md missing | SPIRIT.md missing |
| **Mutation** | Replace content between markers | Overwrite whole file |

The skill shape (per D-016 — each skill carries its own `references/`), the abort-on-missing-source stance, and the diff-then-overwrite flow are identical. The semantic difference is the source of truth: refresh-persona pulls from a bundled file (the persona is universal); refresh-spirit pulls from the user's answers to bundled prompts (the spirit is project-unique).

## Why abort on missing SPIRIT.md (not author from nothing)

Authoring SPIRIT.md from nothing is `write-brief`'s job. Splitting authoring and refreshing into separate surfaces keeps each surface narrow and predictable: `write-brief` on the first brief authors SPIRIT.md inline alongside the brief; `refresh-spirit` only ever updates existing content.

If `refresh-spirit` were to also handle the from-nothing case, two surfaces would compete for first-author responsibility, and the user couldn't predict which would run. The chosen contract — refuse with a clear pointer to `write-brief` (or to manual answer-the-prompts-directly) — costs the user one extra command on the rare projects where `refresh-spirit` is invoked before `write-brief` ever runs, and zero ambiguity in the implementation.

Same shape of decision as refresh-persona's "missing markers → abort with guidance" stance (D-027 part b).

## Why duplicate spirit-prompts.md

`write-brief` and `refresh-spirit` both bundle their own copy of `spirit-prompts.md`. Per D-016 each skill is self-contained — no skill reaches into a sibling's `references/`. The two copies are byte-identical at ship time but may drift if one is edited and the other isn't.

D-016 explicitly accepts this duplication: skill independence outweighs deduplication. The two files reach existing projects via the same plugin-update path, so propagation is uniform — drift only happens if a future commit touches one and not the other. If drift becomes a real problem, a successor brief can introduce a sync mechanism. For v0.9.0, drift is tolerated and not policed.

## What the skill does NOT do

Out of scope per the brief:

- **Authoring SPIRIT.md from nothing** — that's `write-brief`'s job on the first brief in a project.
- **Per-section preservation on overwrite** — whole-file replacement is the contract. Hand-edits to SPIRIT.md outside the skill are forfeit on refresh, by design.
- **Backup of the prior file** — git is the version control system; a `.bak` would be redundant.
- **`--dry-run` flag** — the diff-then-write flow already provides visibility before the mutation.
- **Schema validation of SPIRIT.md** — the file is human prose. Validation = "exists with content."
- **Sync mechanism between write-brief's and refresh-spirit's spirit-prompts.md** — drift tolerated per D-016.
- **Migration tool for projects predating CB-004** — the workbench is the only such project; it gets dogfooded by hand as part of CB-004.

## Workflow (summary)

1. Locate `/code/SPIRIT.md`. If missing → abort with write-brief guidance.
2. Read bundled `references/spirit-prompts.md`. If missing → broken plugin install.
3. Read current SPIRIT.md; parse loosely to extract the existing answer for each of the 8 prompts.
4. Walk the 8 prompts one at a time, pre-filled with current answers. User can keep / edit / replace / set to `n/a`.
5. Compose new SPIRIT.md (single-line `n/a` form if all 8 are `n/a`; otherwise one `## <prompt name>` heading per non-`n/a` answer).
6. Show unified diff. If new content is byte-identical to old → exit silently with "already up to date."
7. Overwrite SPIRIT.md whole-file.
8. Confirm with file path + counts.

## CB-006 evolution (v0.11.0+, D-030) — spirit-signals integration

The skill became **wiki-aware** in v0.11.0. Two new steps insert into the existing flow without changing the existing contract:

**Step 3.5 — Read pending spirit-signals.** Between current-content read (Step 3) and the prompt walk (Step 4), the skill reads `/wiki/spirit-signals.md` if present and parses any `## YYYY-MM-DD — <title>` sections whose footer is `_Status: pending synthesis._`. Already-synthesized signals are skipped (history, not input). Pending signals are surfaced to the user before the walk and **fold into the pre-fill** for each of the 8 prompts. Claude judges which signals are relevant to which prompt — no fixed mapping.

**Step 7.5 — Mark signals synthesized.** After the SPIRIT.md write succeeds, each previously-pending signal's footer is updated from `_Status: pending synthesis._` to `_Status: synthesized into SPIRIT.md on YYYY-MM-DD via /yuval-core:refresh-spirit._`. **Append-only on the signals file** — never delete a signal, never edit body text. Only the footer line is mutated. The full signal history + when each was folded in is the audit trail.

**Sparse-wiki fallback:** missing `/wiki/spirit-signals.md` or no pending entries → existing flow runs unchanged. Wiki read is additive, not gating. Same shape as `write-brief`'s sparse-wiki fallback (D-030).

**Byte-identical SPIRIT.md case:** if Step 6's diff shows no changes (user kept all answers), Step 7.5 still runs to mark pending signals synthesized — keeping all answers is itself a confirmation that the existing synthesis covers the new signals. Step 7 (the SPIRIT.md write) is skipped, but Step 7.5 (the signals update) runs.

**Skill-wiki contract** (declared in the SKILL.md header comment):
- **Reads:** `/code/SPIRIT.md`, `/wiki/spirit-signals.md`
- **Writes:** `/code/SPIRIT.md` (canonical, whole-file, user-confirmed via diff), `/wiki/spirit-signals.md` (status-line append only)
- **Sparse-wiki fallback:** missing file or no pending entries → existing flow unchanged.

This is the consumption side of the workshop layer that CB-005 introduced (`spirit-signals.md` as the raw-signal capture surface). The pattern: **capture raw signals between syntheses (no work at capture time), synthesize on demand (user-confirmed)**. Mirrors the write-brief / promote-to-code pattern where intent is captured at one step and acted on at another, with an explicit hand-off contract between them.

## Cross-references

- **ADR:** [D-028](../../code/DECISIONS.md#d-028) — Project SPIRIT contract.
- **Brief:** [`CB-004`](../../output/briefs/CB-004-spirit-contract.md).
- **Source page (ideation):** [source-framework-feedback-2026-05-03](../../wiki/pages/source-framework-feedback-2026-05-03.md) — Mira framework feedback, the rationale for the contract.
- **Sibling skill:** [refresh-persona](../../wiki/pages/skill-refresh-persona.md) — same skill shape, same abort-on-missing stance, same diff-then-overwrite flow at the persona altitude.
- **Related ADRs:** [D-016](../../code/DECISIONS.md#d-016) (each skill carries its own `references/`; cross-skill drift tolerated), [D-026](../../code/DECISIONS.md#d-026) / [D-027](../../code/DECISIONS.md#d-027) (refresh-persona pattern this skill mirrors).
- **Pattern:** [concept-spirit-contract](concept-spirit-contract.md) — project-vs-brief altitude rule for atmosphere.
