# yuval-core

Core workspace plugin for Yuval's Claude setup.

## What's in it

- **`init` skill** — Scaffolds a new project with four folders (`input/`, `in-progress/`, `output/`, `wiki/`), writes `CLAUDE.md` from the canonical template, injects persona, and initializes the wiki (Karpathy LLM Wiki pattern). The scaffolded wiki includes `index.md`, `log.md`, and `next.md` — the last file carries session-to-session continuity: every session ends by overwriting `next.md` with the current hand-off and printing it as the closing message, and every session starts by reading it.

## Framework principles

Core design commitments across this plugin. Read these before changing `init` or adjacent skills — they exist because earlier iterations rebuilt the wrong thing.

- **Session hand-off lives in `/wiki/next.md`.** Single file, overwritten each session, printed as the closing message of the session ("hot swap"). Do **not** use a `## Next Actions` section inside `index.md` — that pattern was tried and replaced.
- **CLAUDE.md additive merge is strict.** On re-run, the `init` skill never removes or modifies existing lines — only appends. Section renames or removals go through the deprecated-sections mechanism: see `skills/init/references/deprecated-sections.md` and Step 5.5 of the skill.
- **Persona refresh is a deliberate act.** Init never touches content between `<!-- PERSONA:START -->` and `<!-- PERSONA:END -->`. Use `/refresh-persona` when it lands.

## How it reads the marketplace

The `init` skill reads two files bundled with the skill at `skills/init/references/`:

- `claude-template.md` — the project CLAUDE.md template
- `persona.md` — the persona block that gets injected into `CLAUDE.md`

Edits happen in the plugin's git repo; updates propagate through normal plugin updates. If either file is missing at runtime, `init` treats it as a broken install and stops.

## Planned additions

- `llm-wiki` skill — owns `ingest`, `query`, `lint` operations over `/wiki/`
- `init-code` command — scaffolds the `code/` git repo when a project moves to implementation
- `promote-to-code` command — moves an `output/` doc into `code/wiki/` as an ADR
- `refresh-persona` command — re-syncs the persona block in an existing project's `CLAUDE.md` from the skill's bundled `persona.md`

## Install

```
/plugin marketplace add ~/claude-marketplace
/plugin install yuval-core@yuval-marketplace
```
