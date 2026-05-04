# yuval-core

Core workspace plugin for Yuval's Claude setup. Two-mode workspace: ideation at the project root, optional code workspace under `/code/`.

## What's in it

### Ideation (always)

- **`init` skill** — Scaffolds a new project with the standard folders (`input/`, `in-progress/`, `output/`, `output/briefs/`, `wiki/`, `wiki/pages/`, `wiki/ideation/`), writes `CLAUDE.md` from the canonical template, injects persona, and initializes the **three-layer wiki** (see *The three-layer wiki* below). Every artifact is create-if-missing; re-runs are strictly additive.
- **`refresh-persona` skill** — Re-syncs the persona block in an already-initialized project's `CLAUDE.md` from the skill's bundled `persona.md`, without re-running `init`. Replaces only the content between `<!-- PERSONA:START -->` and `<!-- PERSONA:END -->`; aborts with guidance if the markers aren't present.

#### The three-layer wiki (v0.10.0+, D-029)

The wiki is a **workshop**, not a single source of truth. Canonical artifacts (`SPIRIT.md`, briefs in `/output/briefs/`, ADRs in `/code/DECISIONS.md`, code) remain authoritative. The wiki is the surface where intent and atmosphere accumulate before they become commitments. Three layers, each with a distinct role and read cadence:

**1. Generative — *what's coming*. Loaded into context every session.**

| File | Role |
|---|---|
| `wiki/vision.md` | North star. Free prose. Confirmation gate before overwrite. |
| `wiki/goals.md` | Current-phase goals, with target briefs. Active / Closed sections. |
| `wiki/spirit-signals.md` | Raw atmospheric / voice / tone signals captured between `refresh-spirit` runs. **No synthesis at write time** — `refresh-spirit` synthesizes later. |
| `wiki/backlog.md` | Parked tasks ("we should also someday…"). Becomes a brief only via explicit `write-brief`. |
| `wiki/ideation/` | Half-formed ideas. One file per spitball: `idea-<slug>.md`. |

**2. Retrospective — *what happened*. Read on demand.**

`wiki/log.md` (append-only chronological record), `wiki/pages/` (one page per source / entity / concept / decision / session).

**3. State — *current pointers*. Loaded into context every session.**

`wiki/index.md` (catalog), `wiki/next.md` (single-file session hand-off), `wiki/memory-protocol.md` (the protocol itself).

**Proactive generative triggers** (alongside the existing retrospective triggers):

| Trigger | Action |
|---|---|
| User articulates vision / restates "why we're doing this" | Read `vision.md`; propose update; confirm before overwriting. |
| User sets or shifts a phase goal | Append to `goals.md` under `## Active` (with set-date and target brief). |
| User expresses an atmospheric / voice / tone preference | Append to `spirit-signals.md` as a dated raw note (status: pending synthesis). |
| User says "we should also someday…" / "park this" | Append to `backlog.md` as a checkbox item. |
| User spitballs an unfinished idea | Create a new file in `ideation/` named `idea-<slug>.md`. |

**Read-side change.** Session-start sequence is now: `protocol → index → next → vision → goals → spirit-signals → scan input/`. The three generative reads load **project character** into context every session, not just retrospective state.

The wiki ↔ skills contract is one-directional and human-confirmed: skills read the generative layer; skills may modify the wiki; **skills never auto-modify canonical artifacts**. Synthesis (`refresh-spirit`, brief drafting, promotion) is always user-confirmed.

### Code workspace (opt-in, v0.6.0+)

- **`init-code` skill** — Scaffolds `/code/` as a git repo with its own `CLAUDE.md`, `README.md`, `DECISIONS.md`, `.gitignore`, and a self-contained code wiki. Detects an existing `/code/.git/` (e.g., a prior clone) and additively scaffolds without re-running `git init`.
- **`write-brief` skill** — Authors a code brief (`CB-XXX`) interactively. On the first brief in a project (when `/code/SPIRIT.md` is absent), walks 8 spirit prompts inline before Goal and writes SPIRIT.md. On subsequent briefs, asks one question for any per-brief spirit adjustment (default: no). Then walks goal → scope → out-of-scope → references → acceptance. Briefs are the only promotion-eligible artifact.
- **`refresh-spirit` skill** (v0.9.0+) — Re-walks the 8 spirit prompts pre-filled with the project's current `/code/SPIRIT.md` content, shows a diff, overwrites. Mirrors `refresh-persona` (D-026/D-027): aborts cleanly when SPIRIT.md is absent. Use when project atmosphere has evolved enough that a per-brief override would balloon past one paragraph.
- **`/yuval-core:promote-to-code CB-XXX`** — Handshake command (ideation → code). Adds an ADR to `/code/DECISIONS.md`, freezes the brief in place, logs both wikis. Refuses non-briefs, already-promoted briefs, and projects where `/code/SPIRIT.md` is missing entirely (per D-028 — spirit must precede promotion).
- **`/yuval-core:report-back <finding>`** — Reverse handshake (code → ideation). Logs both wikis; optionally touches a `/wiki/pages/` page when the finding is durable.

#### The SPIRIT contract (v0.9.0+, D-028)

`/code/SPIRIT.md` is the project-level single source of truth for atmosphere — interaction shape, voice/tone, anti-shape, silent state, tactile contract, default posture, the one moment, plus a reference scenario. The 8 prompts are bundled as `references/spirit-prompts.md` inside both `write-brief` and `refresh-spirit`.

- **Authored once.** First `write-brief` invocation in a project walks all 8 prompts inline, writes `/code/SPIRIT.md`, then continues with the brief.
- **Refreshed deliberately.** `refresh-spirit` is the dedicated path for evolving SPIRIT.md after authoring.
- **Briefs reference, don't redefine.** The brief template's `## Spirit & Texture` section defaults to `Inherits /code/SPIRIT.md.` Per-brief overrides are a single optional paragraph.
- **Promotion gate.** `promote-to-code` refuses if `/code/SPIRIT.md` is missing. Pure-infrastructure projects with no user surface answer all 8 prompts `n/a` — the resulting one-line SPIRIT.md (`n/a — pure infrastructure project. ...`) passes the gate.

The contract addresses the failure mode where a brief encodes capabilities + acceptance but not interaction shape, and a coding agent fills the gap with the most familiar wrong UX shape (e.g., a form-first UI for a voice-first product). Spirit must be transmitted explicitly across the ideation→code boundary.

#### Recommended app-side pre-flight (not shipped — D-028 part e)

Per D-028, yuval-core does not ship coding-agent persona or pre-flight logic into application `/code/CLAUDE.md` files via `init-code` or any other skill — that lane belongs to the application. The following pattern is **recommended** for an application's coding-agent instructions to adopt voluntarily:

> Before writing user-facing code, locate the project's `/code/SPIRIT.md`. Restate the **Interaction Shape** and one sentence of the **Reference Scenario** back to the user as part of the implementation plan. List three UI patterns (drawn from **Anti-Shape**) the agent will NOT use. If voice/tone samples are absent and the product has an agent surface, ask for at least three before writing copy.

This pre-flight is the smallest item that would have caught the Mira-project misfire (a "voice-first ambient agent" brief that was implemented as a form-first UI). Each application decides whether to adopt it; yuval-core stays out of the application's coding behavior.

## Framework principles

Core design commitments across this plugin. Read these before changing `init` or adjacent skills — they exist because earlier iterations rebuilt the wrong thing.

- **Session hand-off lives in `/wiki/next.md`.** Single file, overwritten each session, printed as the closing message of the session ("hot swap"). Do **not** use a `## Next Actions` section inside `index.md` — that pattern was tried and replaced.
- **CLAUDE.md additive merge is strict.** On re-run, the `init` and `init-code` skills never remove or modify existing lines — only append. Section renames or removals go through the deprecated-sections mechanism: see `skills/init/references/deprecated-sections.md` and Step 5.5 of the skill.
- **Memory protocol lives in the wiki, not in CLAUDE.md.** Per D-018, `/wiki/memory-protocol.md` (and `/code/wiki/memory-protocol.md`) carry the wiki/session-continuity rules. Both are populated from the same plugin reference (`skills/init/references/memory-protocol.md`) so ideation and code stay synchronized via re-runs.
- **Briefs are the only cross-mode promotion artifact.** Specs, reports, and design docs in `/output/` are reference material. Briefs in `/output/briefs/` are commitments. Only briefs go through `promote-to-code` and become ADRs in `/code/DECISIONS.md`. See D-023.
- **Handshake commands are the only sanctioned cross-mode edges.** `promote-to-code` (ideation → code) and `report-back` (code → ideation) are the only two paths that mutate state across the boundary. Everything else stays within its own mode. See D-024.
- **Persona refresh is a deliberate act.** Init never touches content between `<!-- PERSONA:START -->` and `<!-- PERSONA:END -->`. Use `/yuval-core:refresh-persona` to pull updates into existing projects.
- **Spirit lives at the project level, not per brief (D-028).** `/code/SPIRIT.md` is the project's single source of truth for atmosphere. `write-brief` authors it on the first brief in a project; `refresh-spirit` evolves it; briefs reference it with at most a one-paragraph override. `promote-to-code` refuses on missing SPIRIT.md. Application coding-agent persona is the application's lane, not yuval-core's.
- **The wiki is a workshop, not a single source of truth (D-029).** Three layers — generative (vision / goals / spirit-signals / backlog / ideation), retrospective (log + pages), state (index / next / memory-protocol). Skills consume the generative layer; canonical artifacts (SPIRIT.md, briefs, ADRs, code) remain authoritative and are never auto-mutated. Synthesis is always user-confirmed.

## How it reads the marketplace

Each skill reads its own bundled references from `skills/<skill>/references/`. Per D-016, no skill reaches into a sibling's folder — exception: `init-code` reads `memory-protocol.md` from `skills/init/references/` so ideation and code wikis share a single canonical copy of the protocol.

If any required reference is missing at runtime, the skill treats it as a broken install and stops.

## Planned additions

- `llm-wiki` skill — owns `ingest`, `query`, `lint` operations over `/wiki/` (and a `/code/wiki/` mirror once the surface stabilizes).

## Install

```
/plugin marketplace add yuvallavi2/yuval-claude-marketplace
/plugin install yuval-core@yuval-claude-marketplace
```
