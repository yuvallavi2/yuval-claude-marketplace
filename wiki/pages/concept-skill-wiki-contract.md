# Concept — Skill-wiki contract (D-030)

## The three rules

1. **Read freely.** Skills may read any wiki file as context or pre-fill. No special permission, no quotas. The generative layer (vision, goals, spirit-signals, backlog, ideation) in particular is *meant* to be loaded into skill context.
2. **Write narrowly.** Skills may only write to the wiki files they explicitly declare in their `SKILL.md` (or command file). The declared write set is part of the contract — extending it requires a brief.
3. **Never auto-modify canonical artifacts from wiki content alone.** Wiki content is *input* to a synthesis; the synthesis itself is user-confirmed. Canonical artifacts: `/code/SPIRIT.md`, briefs in `/output/briefs/`, ADRs in `/code/DECISIONS.md`, code files, persona.

These rules sit one altitude above any individual skill — they are the constraint that lets the wiki and the skills evolve independently without either growing tendrils into the other.

## Why this contract exists

CB-005 (D-029) introduced the workshop layer, giving the wiki a generative half (vision / goals / spirit-signals / backlog / ideation) for skills to consume. CB-006 wires the consumption side. Without an explicit contract:

- **Read pollution.** Every skill could grow ad-hoc reads from any wiki file, making the dependency graph between skills and wiki opaque.
- **Write pollution.** Skills could mutate wiki state as a convenience, eroding the audit trail and producing surprise edits across runs.
- **Canonical drift.** Skills could update SPIRIT.md or briefs from wiki content directly, removing the user from the loop and producing canonical artifacts that don't match what the user actually approved.

The contract addresses each: declared reads make dependencies legible (they live in the SKILL.md header comment); declared writes constrain the surface area; the never-auto-modify-canonical rule keeps synthesis user-confirmed.

## Wiki-aware skills (the MVP)

| Skill | Reads | Writes (wiki) | Sparse-wiki fallback |
|---|---|---|---|
| `write-brief` | `/wiki/goals.md`, `/wiki/backlog.md`, `/wiki/ideation/*.md`, `/code/SPIRIT.md` | none directly — stashes `<!-- backlog-closes: ... -->` and `<!-- ideation-promotes: ... -->` markers in the brief for `promote-to-code` | each read best-effort; missing/malformed → skip step silently |
| `refresh-spirit` | `/wiki/spirit-signals.md` (pending entries only), `/code/SPIRIT.md` | `/wiki/spirit-signals.md` (status-line append only) | missing file or no pending → existing flow unchanged |
| `promote-to-code` | brief markers; `/wiki/backlog.md` (only with marker); `/wiki/ideation/*.md` (only with marker) | `/wiki/backlog.md` (close marked items), `/wiki/ideation/archive/*.md` (move marked files in) | marker absent → side-effect skipped silently |

## Skills deliberately staying wiki-blind

- **`init`, `init-code`** — these are bootstrap skills. They *create* the wiki; they don't consume it. Reading from a wiki that doesn't yet exist is a non-question. (Out of scope per CB-006.)
- **`refresh-persona`** — persona is project-agnostic (one persona for everyone), but the wiki is project-specific. Reading the project wiki to refresh a global artifact would conflate altitudes. Persona refresh is purely a bundled-source-to-CLAUDE.md operation, no wiki involvement. (D-030 part c.)

If a future skill emerges with a real need to read a wiki file beyond the MVP set, the contract is extended via a brief — not by the skill quietly growing a new read.

## Brief-marker convention

Two HTML-comment markers carry user-confirmed wiki side-effects from `write-brief` to `promote-to-code`:

```markdown
# CB-XXX — Title

_Created: YYYY-MM-DD | Status: open | ADR: TBD_
<!-- backlog-closes: <slug1>,<slug2> -->
<!-- ideation-promotes: <slug-a>,<slug-b> -->

## Goal
...
```

| Marker | Authored by | Consumed by | Effect at promotion |
|---|---|---|---|
| `<!-- backlog-closes: ... -->` | `write-brief` Step 1.8 (LLM-judged matches, user-confirmed) | `promote-to-code` Step 5.5 | Strike matching items in `/wiki/backlog.md` and move from `## Active` to `## Closed`. Append closure annotation pointing back at the brief. |
| `<!-- ideation-promotes: ... -->` | `write-brief` Step 3.5 (user names ideation files in References) | `promote-to-code` Step 5.6 | Move `/wiki/ideation/idea-<slug>.md` to `/wiki/ideation/archive/idea-<slug>.md` with a footer linking back to the brief. |

Either marker is omitted when its list is empty. Briefs without markers cause no wiki side-effects at promotion. The pre-v0.11.0 promotion contract is exactly the no-markers case; CB-006 is purely additive.

## Why markers, not promotion-time matching

The user-confirmed match between brief and backlog/ideation happens at **brief-write time** in `write-brief`. By the time the brief reaches promotion (often days later), the slugs are already user-approved and frozen in the brief. Promotion just executes the contract.

Two reasons this is the right shape:

1. **Synthesis stays where context is freshest.** The user is actively authoring the brief; backlog and ideation items are visible in the same conversation. By promotion, the conversation may be days later — re-asking the user to match items would impose load and risk worse matches.
2. **Promotion stays mechanical and auditable.** The command never makes a judgment call. Either a marker is present (act on it) or absent (no-op). The dependency graph between brief author and promotion runner is one-directional.

This mirrors the spirit-signal flow: `spirit-signals.md` captures raw signals throughout the project's lifetime (no synthesis at write time); `refresh-spirit` synthesizes pending signals into SPIRIT.md when the user invokes it (user-confirmed via diff). The pattern: **capture intent at one step, act on it at another, with an explicit hand-off contract between them.**

## Out of scope (rejected variants)

- **Auto-promotion of backlog items into briefs.** A backlog item becomes a brief only via explicit `write-brief` invocation. No nagging, no scheduled review.
- **Fuzzy-string or tag-based matching.** Match is LLM-judged with user confirmation. Robust to paraphrase, fails gracefully when nothing fits.
- **Cross-project wiki reads.** Skills read the local project's `/wiki/` only. No reaching across project boundaries.
- **Schema validation of wiki files.** All wiki files remain human prose. The format conventions are conventions, not enforced schemas. Malformed file → fall back to the sparse path.
- **`--no-wiki` flag on any skill.** If the user wants to skip the wiki read, they delete or rename the file temporarily. Adding a flag is feature creep.
- **Reading `vision.md` or `open-questions.md` in any skill.** Vision is too unstructured for skills to consume reliably; open-questions are the user's deliberation surface, not skill input.
- **Wall-of-recent-signals dumps in prompts.** Each skill surfaces only what's load-bearing for the current prompt.

## References

- [D-030 — Wiki-aware skills (CB-006)](../../DECISIONS.md#d-030)
- [D-029 — Wiki 2.0: workshop layer (CB-005)](../../DECISIONS.md#d-029) — defines the wiki surfaces this contract reads.
- [D-016 — Each skill carries its own references/](../../DECISIONS.md#d-016) — still respected; the wiki is the user's surface, not a sibling skill's `references/`.
- [D-023 — Briefs are the only commitment artifact](../../DECISIONS.md#d-023) — markers don't change this; they just attach wiki side-effects to existing promotion events.
- [D-024 — Handshake commands are the only sanctioned cross-mode edges](../../DECISIONS.md#d-024) — wiki-aware behavior is added to `promote-to-code`, which remains a handshake.
- [CB-006 brief](../../../output/briefs/CB-006-wiki-aware-skills.md)
- Sibling pages: [`concept-wiki-workshop.md`](concept-wiki-workshop.md), [`skill-write-brief.md`](skill-write-brief.md), [`skill-refresh-spirit.md`](skill-refresh-spirit.md), [`command-promote-to-code.md`](command-promote-to-code.md).
