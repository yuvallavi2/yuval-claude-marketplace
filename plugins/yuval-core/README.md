# yuval-core

Core workspace plugin for Yuval's Claude setup. Two-mode workspace: ideation at the project root, optional code workspace under `/code/`.

## What's in it

### Ideation (always)

- **`init` skill** — Scaffolds a new project with five folders (`input/`, `in-progress/`, `output/`, `output/briefs/`, `wiki/`), writes `CLAUDE.md` from the canonical template, injects persona, and initializes the wiki (Karpathy LLM Wiki pattern). The scaffolded wiki includes `memory-protocol.md` (the framework-canonical memory protocol), `index.md`, `log.md`, and `next.md` — the last file carries session-to-session continuity: every session ends by overwriting `next.md` with the current hand-off and printing it as the closing message, and every session starts by reading it.
- **`refresh-persona` skill** — Re-syncs the persona block in an already-initialized project's `CLAUDE.md` from the skill's bundled `persona.md`, without re-running `init`. Replaces only the content between `<!-- PERSONA:START -->` and `<!-- PERSONA:END -->`; aborts with guidance if the markers aren't present.

### Code workspace (opt-in, v0.6.0+)

- **`init-code` skill** — Scaffolds `/code/` as a git repo with its own `CLAUDE.md`, `README.md`, `DECISIONS.md`, `.gitignore`, and a self-contained code wiki. Detects an existing `/code/.git/` (e.g., a prior clone) and additively scaffolds without re-running `git init`.
- **`write-brief` skill** — Authors a code brief (`CB-XXX`) interactively. Walks goal → scope → out-of-scope → references → acceptance. Briefs are the only promotion-eligible artifact.
- **`/yuval-core:promote-to-code CB-XXX`** — Handshake command (ideation → code). Adds an ADR to `/code/DECISIONS.md`, freezes the brief in place, logs both wikis. Refuses non-briefs and already-promoted briefs.
- **`/yuval-core:report-back <finding>`** — Reverse handshake (code → ideation). Logs both wikis; optionally touches a `/wiki/pages/` page when the finding is durable.

## Framework principles

Core design commitments across this plugin. Read these before changing `init` or adjacent skills — they exist because earlier iterations rebuilt the wrong thing.

- **Session hand-off lives in `/wiki/next.md`.** Single file, overwritten each session, printed as the closing message of the session ("hot swap"). Do **not** use a `## Next Actions` section inside `index.md` — that pattern was tried and replaced.
- **CLAUDE.md additive merge is strict.** On re-run, the `init` and `init-code` skills never remove or modify existing lines — only append. Section renames or removals go through the deprecated-sections mechanism: see `skills/init/references/deprecated-sections.md` and Step 5.5 of the skill.
- **Memory protocol lives in the wiki, not in CLAUDE.md.** Per D-018, `/wiki/memory-protocol.md` (and `/code/wiki/memory-protocol.md`) carry the wiki/session-continuity rules. Both are populated from the same plugin reference (`skills/init/references/memory-protocol.md`) so ideation and code stay synchronized via re-runs.
- **Briefs are the only cross-mode promotion artifact.** Specs, reports, and design docs in `/output/` are reference material. Briefs in `/output/briefs/` are commitments. Only briefs go through `promote-to-code` and become ADRs in `/code/DECISIONS.md`. See D-023.
- **Handshake commands are the only sanctioned cross-mode edges.** `promote-to-code` (ideation → code) and `report-back` (code → ideation) are the only two paths that mutate state across the boundary. Everything else stays within its own mode. See D-024.
- **Persona refresh is a deliberate act.** Init never touches content between `<!-- PERSONA:START -->` and `<!-- PERSONA:END -->`. Use `/yuval-core:refresh-persona` to pull updates into existing projects.

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
