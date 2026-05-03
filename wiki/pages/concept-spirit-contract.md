# Concept — Spirit contract

_Origin: CB-004 / D-028 — shipped in v0.9.0 (2026-05-03). Source rationale: [source-framework-feedback-2026-05-03](../../wiki/pages/source-framework-feedback-2026-05-03.md) (Mira project)._

## What it solves

A brief that lists capabilities and acceptance criteria, but no interaction shape, defaults the coding agent toward the most familiar wrong UX shape. Mira shipped a brief for a "voice-first ambient agent" and the coding agent built a form-first UI — capabilities satisfied, soul lost.

The fix isn't more capability detail. The fix is encoding **how the product must feel** alongside **what it must do**.

## The altitude rule

Atmosphere is set **once per project**, not per brief.

```
PROJECT LEVEL                                 BRIEF LEVEL
─────────────                                 ───────────
/code/SPIRIT.md   ← authored ONCE             CB-XXX brief
  • Reference Scenario                          • Goal
  • Interaction Shape                           • Scope
  • Voice / Tone samples                        • Out of scope
  • Anti-Shape                                  • Acceptance
  • Silent State                                • Spirit & Texture:
  • Tactile Contract                              "Inherits /code/SPIRIT.md"
  • Default Posture                                 (or one-paragraph override)
  • The One Moment

(re-walk via /yuval-core:refresh-spirit if it ever needs to evolve)
```

Two earlier framings were rejected before landing here:

1. **Pre-flight checklist shipped via `init-code`.** Wrong altitude — that crosses the framework-vs-application boundary. Yuval-core is a framework of work; the application's `/code/CLAUDE.md` is app-owned territory. Pre-flight is documented as a *recommended* app-side pattern in the README, not auto-injected by any skill.
2. **Spirit & Texture authored per brief.** Wrong altitude in the other direction — would force the same eight answers across every brief in the project, redundantly. The atmosphere of a project is one thing.

The altitude landed at: **project-level SPIRIT.md as single source of truth; briefs reference it; per-brief overrides are an opt-in single paragraph.**

## The `n/a-is-valid` pattern

Not every project has user-facing surface. Frameworks, CLIs, build tooling, libraries — pure infrastructure projects with no atmosphere to transmit.

For those, every one of the 8 spirit prompts is answerable as `n/a`. The resulting SPIRIT.md is a single line:

```
n/a — pure infrastructure project. No user surface, no atmosphere to transmit.
```

This still passes the `promote-to-code` SPIRIT.md gate. The gate is presence-of-file, not presence-of-content. Promotion is allowed; the brief's acceptance criteria carry the full intent.

The yuval-core workbench itself is the inaugural example of this pattern — `/code/SPIRIT.md` was authored as part of CB-004's dogfood step with exactly this content.

The alternative (only require SPIRIT.md when the project has user surface) was rejected because *whether the project has user surface* is itself a judgment call that benefits from being made explicitly once and recorded — answering all 8 prompts `n/a` is the act of making that judgment.

## Where the contract enforces itself

The contract has four enforcement points:

| Surface | Enforcement |
|---|---|
| `write-brief` (first invocation in project) | Walks the 8 prompts inline before Goal; writes `/code/SPIRIT.md`; continues. |
| `write-brief` (subsequent invocations) | Asks one question — *"Any spirit adjustment for this brief? (default: no)"* — and inherits otherwise. |
| `refresh-spirit` | Mirrors `refresh-persona`: re-walks prompts pre-filled, diff, overwrite. Aborts cleanly if SPIRIT.md is absent. |
| `promote-to-code` | Refuses to promote any brief if `/code/SPIRIT.md` is missing entirely. |

The promote-to-code gate is the load-bearing one — it ensures spirit precedes any commitment to code. The other three are how spirit gets there.

## Why presence-only validation, not schema

SPIRIT.md is human prose. Each of the 8 prompts is a prompt for narrative, not a structured field. Trying to enforce a schema (YAML/JSON fields, regex on heading levels, etc.) would deform the prose and add zero detection of actual deficits — a project can satisfy a schema with empty stubs and still have no atmosphere encoded.

The chosen validation is presence-of-file. A user who writes a one-line SPIRIT.md saying *"the product feels like a chair"* technically passes — but they've made an explicit choice, on the record, that's worse than a missing file because there's no ambiguity that they meant exactly that. The schema-less approach pushes accountability to the human; structured validation only pushes it to a machine that can be fooled.

## How this differs from the refresh-persona altitude

Persona (D-006/D-007/D-026) and spirit (D-028) are both framework-canonical concepts encoded in skills, but they sit at different altitudes:

- **Persona** is about working *with Yuval*. It's universal across projects, bundled in the plugin, refreshable into existing projects via `refresh-persona`. One persona file, every project gets a copy.
- **Spirit** is about a *specific product's* atmosphere. It's project-unique, written by answering 8 prompts, refreshable via `refresh-spirit`. One SPIRIT.md per project, content varies wildly across projects.

Both are persistent agent context. Both have refreshable bundled-source-or-prompts surfaces. The semantic difference: the persona is *Yuval's* invariant; the spirit is each *project's* invariant.

## Cross-references

- **ADR:** [D-028](../../code/DECISIONS.md#d-028) — Project SPIRIT contract.
- **Brief:** [`CB-004`](../../output/briefs/CB-004-spirit-contract.md).
- **Module page:** [skill-refresh-spirit](skill-refresh-spirit.md) — the skill that mirrors the refresh-persona pattern.
- **Source page (ideation):** [source-framework-feedback-2026-05-03](../../wiki/pages/source-framework-feedback-2026-05-03.md) — Mira framework feedback, the originating rationale.
- **Related ADRs:** [D-006](../../code/DECISIONS.md#d-006) (persona scope: Cowork-only), [D-007](../../code/DECISIONS.md#d-007) (persona contract & marker pair), [D-016](../../code/DECISIONS.md#d-016) (each skill carries its own `references/`), [D-023](../../code/DECISIONS.md#d-023) (briefs are the only commitment artifact — SPIRIT.md sits one altitude up but is reachable from every brief), [D-026](../../code/DECISIONS.md#d-026) / [D-027](../../code/DECISIONS.md#d-027) (refresh-persona pattern, the precedent for refresh-spirit).
