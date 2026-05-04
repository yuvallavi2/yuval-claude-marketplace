# Concept — Wiki as workshop (D-029, CB-005)

## Stance

The wiki is a **workshop**, not a single source of truth. Canonical artifacts (`/code/SPIRIT.md`, briefs in `/output/briefs/`, ADRs in `/code/DECISIONS.md`, code) hold the authoritative truth. The wiki is the surface where intent, atmosphere, and signals **accumulate before they get committed** to those canonical artifacts.

Skills consume the wiki as input. Skills may modify the wiki (capture, append, status updates). **Skills never auto-modify canonical artifacts.** Synthesis from wiki signals into a canonical artifact is always user-confirmed (`refresh-spirit`, `write-brief`, `promote-to-code`).

## Three layers

| Layer | Files | Read cadence | Role |
|---|---|---|---|
| **Generative** | `vision.md`, `goals.md`, `spirit-signals.md`, `backlog.md`, `ideation/` | Every session (loaded into context) | What's coming. Intent, atmosphere, queued work, half-formed ideas. |
| **Retrospective** | `log.md`, `pages/` | On demand | What happened. Durable record. |
| **State** | `index.md`, `next.md`, `memory-protocol.md` | Every session (loaded into context) | Current pointers. Where the project is right now. |

Both ideation and code wikis carry the three layers; the code wiki's generative layer is **opt-in by future briefs** (D-029 ships the layer to ideation-side `init`; whether `init-code` mirrors it is a separate decision and not part of CB-005).

## Why workshop, not single-source-of-truth

The single-source-of-truth alternative was considered and explicitly rejected. Three reasons:

1. **Round-trip churn.** SoT requires every artifact to flow through the wiki. SPIRIT.md, briefs, and ADRs would each need a wiki representation, kept in sync with the canonical file. Two surfaces, one truth — every change is duplicated, every drift is a bug.
2. **Migration cost.** Existing canonical artifacts (CB-001 through CB-004, D-001 through D-028, code) are already authoritative in their canonical locations. SoT would require backfilling all of them into the wiki — high cost, no compensating benefit beyond philosophical purity.
3. **Wrong contract for atmosphere.** Atmosphere signals are inherently **append-only between syntheses** — captured raw, then folded into SPIRIT.md by `refresh-spirit`. The audit trail (raw signals → synthesized result) belongs in the wiki. The truth (current SPIRIT.md) belongs in the canonical file. Forcing both into one surface deforms one or the other.

The workshop model keeps each artifact authoritative in its canonical location and makes the wiki the place where signals accumulate **before** they get committed. Each layer is small, each contract is narrow, and each canonical artifact stays single-source for its own truth.

## Generative-trigger contracts

Each generative file has a precise format, encoded in the bundled templates at `plugins/yuval-core/skills/init/references/wiki-templates/`. The formats exist so CB-006's wiki-aware skills (`write-brief`, `refresh-spirit`, `promote-to-code`) have a stable surface to query.

| File | Format | Read by (CB-006) |
|---|---|---|
| `vision.md` | Free prose | `write-brief` (loads into context for goal/scope framing) |
| `goals.md` | `## Active` / `## Closed` checklists with set-date + target brief | `write-brief` (proposes goals as brief candidates), `promote-to-code` (closes matched goals) |
| `spirit-signals.md` | Dated `## YYYY-MM-DD — title` sections, status footer | `refresh-spirit` (synthesizes pending signals into SPIRIT.md) |
| `backlog.md` | `## Active` / `## Closed` checklists with kebab-slug | `write-brief` (offers backlog items as brief candidates), `promote-to-code` (closes matched backlog items) |
| `ideation/` | Free-form, one file per idea | All three skills may surface relevant ideas as inspiration |

**No auto-synthesis at write time.** The triggers in `memory-protocol.md` capture raw signal and stop. Synthesis happens only when the consuming skill runs and the user confirms.

## Read-side discipline

Session-start read sequence is now: `protocol → index → next → vision → goals → spirit-signals → scan input/`. The three generative reads load **project character** into context every session, not just retrospective state. This is the load-bearing change in D-029 — without it, the new files exist but never reach context, defeating the whole point.

## Out-of-scope (rejected variants)

- Schema validation of generative files. All five are human prose (vision) or human checklists with light conventions (others). Validation = "exists with content"; no parse, no schema check. Mirrors SPIRIT.md's stance from D-028.
- Auto-classification of mid-conversation content. Categories are populated by Claude proactively recognizing trigger patterns, not by ML/heuristic auto-routing.
- Backlog → brief automation. Backlog items become briefs only via explicit `write-brief` invocation by the user. No nagging, no scheduled review.
- Cross-wiki sync between ideation `/wiki/` and code `/code/wiki/`. Each wiki tracks its altitude; the existing log-both-wikis pattern from `promote-to-code` and `report-back` continues unchanged.
- Renaming or restructuring existing retrospective files. `log.md`, `next.md`, `pages/`, `memory-protocol.md` keep their current locations and contracts. CB-005 is **additive**.

## References

- [D-029 — Wiki 2.0: workshop layer (CB-005)](../../DECISIONS.md#d-029-—-wiki-2.0-workshop-layer-cb-005)
- [D-030 — Wiki-aware skills (CB-006)](../../DECISIONS.md#d-030-—-wiki-aware-skills-cb-006) — the consuming side; pending implementation.
- [CB-005 brief](../../../output/briefs/CB-005-wiki-workshop-layer.md) — frozen, immutable post-promotion.
- [CB-006 brief](../../../output/briefs/CB-006-wiki-aware-skills.md) — sibling brief; defines the query contract per skill.
- Bundled templates: `plugins/yuval-core/skills/init/references/wiki-templates/{vision,goals,spirit-signals,backlog,ideation-readme}.md`.
- Memory protocol (canonical): `plugins/yuval-core/skills/init/references/memory-protocol.md`.
