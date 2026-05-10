# Skill — `init-code`

Scaffolds the code workspace (`/code/`) for an already-initialized ideation project: git repo, code-mode `CLAUDE.md`, `README.md`, `DECISIONS.md`, `.gitignore`, and a self-contained code wiki. Refuses if no `CLAUDE.md` exists at the project root — `init-code` is an extension to an already-initialized project, not a replacement for `init` (D-022). Re-runs are strictly additive.

## Current scope (v0.6.0+)

`init-code` produces the following layout under `/code/`:

```
/code/
├── CLAUDE.md          (code-mode rules — additive merge if present)
├── README.md          (create-if-missing)
├── DECISIONS.md       (append-only ADR log — create-if-missing)
├── .gitignore         (create-if-missing)
└── /wiki/             (code-side memory; self-contained per D-021)
    ├── memory-protocol.md   (refreshed from skills/init/references/ on each run — single canonical copy)
    ├── index.md
    ├── log.md
    ├── next.md
    └── pages/
```

The skill detects three modes — `first-run`, `re-run-existing-repo`, `re-run-no-repo` — and adapts (run `git init -b main` only when needed; commit only on first-run; never touch existing git state).

## What changed in v0.12.0 (CB-007, D-031)

The `init-code` skill's bundled templates now emit the **bidirectional wiki reads** protocol on the code side. Two surfaces updated: `code-claude-template.md` gains a "reads are sanctioned" clarifier in the "Handshake commands (cross-mode edges)" section (writes go through handshakes; reads do not); the inline `index.md` template inside `SKILL.md` Step 6 gains a `## Sister wiki` pointer at the top alongside the existing `Current next action` blockquote.

The skill continues to source `memory-protocol.md` from `skills/init/references/` (per D-021's single-canonical-copy rule), so the `## Cross-wiki reads` section that lands in `/code/wiki/memory-protocol.md` is byte-identical to the one in `/wiki/memory-protocol.md`. No skill orchestration logic changed.

## Discipline preserved

- `init-code` aborts if `CLAUDE.md` is missing at the project root (D-022 — separate skill, separate boundary).
- Re-runs on an existing git repo do **not** run `git init` and do **not** make a commit. Diff is left for the user to review and commit per the workbench commit workflow.
- `code-claude-template.md` re-run merge is additive-only — same discipline as ideation `init`.
- `memory-protocol.md` is read from `skills/init/references/` (no copy in `init-code/references/`), preserving the single-canonical-copy invariant.

## References

- Bundled SKILL: [`plugins/yuval-core/skills/init-code/SKILL.md`](../../plugins/yuval-core/skills/init-code/SKILL.md)
- Bundled references: [`plugins/yuval-core/skills/init-code/references/`](../../plugins/yuval-core/skills/init-code/references/) — `code-claude-template.md`, `code-readme-template.md`, `code-decisions-template.md`, `code-gitignore-template`. Note: `code-wiki-templates/` is empty by design — the index/log/next templates live inline in `SKILL.md`; `memory-protocol.md` is read from `skills/init/references/`.
- ADRs: [D-019](../../DECISIONS.md#d-019) (code workspace extension — bootstrap brief), [D-020](../../DECISIONS.md#d-020) (code workspace nested under ideation root), [D-021](../../DECISIONS.md#d-021) (two wikis, shared protocol, each self-contained), [D-022](../../DECISIONS.md#d-022) (`init-code` is a separate skill from `init`), [D-031](../../DECISIONS.md#d-031) (bidirectional wiki reads).
- Brief: [CB-001](../../../output/briefs/CB-001-bootstrap-code-workspace.md) (frozen — original bootstrap), [CB-007](../../../output/briefs/CB-007-bidirectional-wiki-reads.md) (frozen — cross-wiki reads).
