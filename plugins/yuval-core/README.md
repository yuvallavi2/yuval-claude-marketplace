# yuval-core

Core workspace plugin for Yuval's Claude setup.

## What's in it

- **`init` skill** — Scaffolds a new project with four folders (`input/`, `in-progress/`, `output/`, `wiki/`), writes `CLAUDE.md` from the canonical template, injects persona, and initializes the wiki (Karpathy LLM Wiki pattern).

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
