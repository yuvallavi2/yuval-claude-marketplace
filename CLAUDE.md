# CLAUDE.md — Code Workspace (claude-marketplace-dev)

_See [`../CLAUDE.md`](../CLAUDE.md) for project identity, persona, and ideation rules. This file governs code-mode work inside `/code/`._

---

## Identity

- **Project Name:** claude-marketplace-dev
- **Code workspace created:** 2026-04-27 (via the manual workbench bootstrap; `init-code` semantics applied later in CB-001 to scaffold the inner CLAUDE.md and code wiki)
- **Git repo scope:** this folder (`/code/`) and everything beneath it. Ideation lives one level up at `../` and is not in this repo.
- **Special note:** This `/code/` directory is itself the public `yuval-claude-marketplace` plugin repo. Most code workspaces will have a `README.md` at the root; this one's user-facing README lives at `plugins/yuval-core/README.md` to match the plugin's role on GitHub. Do not auto-scaffold a generic top-level README on re-runs of `init-code` — the absence is intentional.

---

## Folder Structure

```
/code/
├── CLAUDE.md          → this file (code-mode rules)
├── DECISIONS.md       → append-only ADR log (D-001…)
├── LICENSE
├── .gitignore
├── .claude-plugin/    → marketplace.json
├── plugins/           → the yuval-core plugin source
└── /wiki/             → code-side long-term memory (Karpathy LLM Wiki pattern)
    ├── memory-protocol.md
    ├── index.md
    ├── log.md
    ├── next.md
    └── pages/
```

The ideation workspace at `../` owns project identity. Ideation artifacts (`../output/`, `../wiki/`) are **read-only** from code-mode — the only sanctioned mutation paths back to ideation are the handshake commands (see below).

---

## Long-term memory protocol

All rules for operating the code wiki, session continuity, and the session hand-off live in `/code/wiki/memory-protocol.md`. Read that file at session start, before any wiki operation. It is the single source of truth for the memory protocol.

---

## Code-mode rules

### Branching
Solo development on `main` (D-015). No feature branches, no PR workflow. When more than one person contributes, revisit.

### Commits
Make commits **continuously** during a brief's implementation — a brief typically spans many commits. Closure is the milestone marker, not the moment work hits the remote.

Workflow per the workbench convention:
1. User edits files (often with Claude's help).
2. User says "commit and push" (or similar).
3. Claude shows the diff, drafts a commit message, commits, pushes.
4. User reviews after the fact.

Commit message style: imperative mood, short summary line (under 72 chars), blank line, optional body explaining *why*.

### ADR discipline
Every structural change → a new `D-xxx` entry in `/code/DECISIONS.md` *before* the change merges. If the change came from a promoted brief, the ADR was already added by `promote-to-code`. If it's a refactor decision discovered mid-implementation, log it manually with operation `adr` in `/code/wiki/log.md`.

### Tags
Two independent tag namespaces (D-024):
- `brief/CB-XXX` — annotated tag at the commit where a brief's acceptance criteria are met. One per closed brief.
- `vX.Y.Z` — release tag, semver. May bundle multiple closed briefs.

Both can coexist on the same commit. Annotate releases with the brief tags they include.

### Plugin-specific commit hygiene
Because this repo IS the plugin's distribution surface:
- Every change to `plugins/yuval-core/` may need a version bump in `.claude-plugin/marketplace.json`.
- Skill or command additions are user-visible — keep `plugins/yuval-core/README.md` and `marketplace.json` description in sync.
- Test in a scratch project before pushing to `main` (per workbench `Final Checklist Before Shipping a New Skill or Command`).

---

## Handshake commands (cross-mode edges)

The only sanctioned mutations across the ideation/code boundary:

- **`/yuval-core:promote-to-code CB-XXX`** — receive a brief from ideation. Adds an ADR to `/code/DECISIONS.md`, freezes the brief, logs both wikis. Refuses non-briefs and already-promoted briefs.
- **`/yuval-core:report-back <finding>`** — push a code-side finding back to ideation. Logs both `/wiki/log.md` and `/code/wiki/log.md`, optionally touches a `/wiki/pages/` page.

Routine commits, code edits, and code-wiki updates **stay within code mode** — they do not need a handshake command. Use the handshake commands only when crossing the boundary.

---

## Standalone-mount note

This `CLAUDE.md` is self-sufficient for a Cowork mount of just `/code/`. In that mode the ideation folder is unreachable, so `report-back` and `promote-to-code` cannot run — code work continues, but cross-mode handshakes are deferred until the project root is mounted again.
