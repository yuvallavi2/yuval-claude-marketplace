# yuval-core

Core workspace plugin for Yuval's Claude setup.

## What's in it

- **`init` skill** — Scaffolds a new project with four folders (`input/`, `in-progress/`, `output/`, `wiki/`), writes `CLAUDE.md` from the canonical template, injects persona, and initializes the wiki (Karpathy LLM Wiki pattern).

## How it reads the marketplace

The `init` skill reads two files at runtime from `~/claude-marketplace/templates/`:

- `claude-template.md` — the project CLAUDE.md template
- `persona.md` — the persona block that gets injected into `CLAUDE.md`

This lets you edit the template or persona in one place, and every future `init` picks up the change.

If either file is missing, `init` falls back to the bundled copy (`skills/init/references/claude-template.md`) and warns in the final confirmation.

## Planned additions

- `llm-wiki` skill — owns `ingest`, `query`, `lint` operations over `/wiki/`
- `init-code` command — scaffolds the `code/` git repo when a project moves to implementation
- `promote-to-code` command — moves an `output/` doc into `code/wiki/` as an ADR
- `refresh-persona` command — re-syncs the persona block in an existing project's `CLAUDE.md` from `~/claude-marketplace/templates/persona.md`

## Install

```
/plugin marketplace add ~/claude-marketplace
/plugin install yuval-core@yuval-marketplace
```
