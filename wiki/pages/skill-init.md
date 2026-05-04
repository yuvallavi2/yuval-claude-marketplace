# Skill — `init`

Scaffolds a new ideation project: folders, `CLAUDE.md`, persona injection, and the wiki. Re-runs are strictly additive (per D-018 and the deprecated-sections channel).

## Current scope (v0.10.0+)

`init` produces a project with:

```
/input/
/in-progress/
/output/
/output/briefs/
/wiki/
  ├── memory-protocol.md       (state layer — canonical, refreshed on every re-run)
  ├── index.md                 (state layer — written from wiki-index-template.md)
  ├── log.md                   (retrospective layer — append-only)
  ├── next.md                  (state layer — single-file session hand-off)
  ├── vision.md                (generative layer — D-029)
  ├── goals.md                 (generative layer — D-029)
  ├── spirit-signals.md        (generative layer — D-029)
  ├── backlog.md               (generative layer — D-029)
  ├── pages/                   (retrospective layer)
  └── ideation/                (generative layer — D-029)
      ├── README.md
      └── .gitkeep
```

Plus `CLAUDE.md` at the project root with persona block injected from `references/persona.md`.

## What changed in v0.10.0 (CB-005, D-029)

The wiki is now a **three-layer workshop**, not a flat retrospective archive. `init` scaffolds the full layer set on first run and back-fills missing artifacts on re-run (additive, never destructive).

**New scaffolded artifacts (Step 1 + Step 6):**

- `/wiki/ideation/` directory (created via `mkdir -p` in Step 1).
- `/wiki/vision.md`, `/wiki/goals.md`, `/wiki/spirit-signals.md`, `/wiki/backlog.md` — written from `references/wiki-templates/{vision,goals,spirit-signals,backlog}.md` with `{{PROJECT_NAME}}` substitution. Each is create-if-missing — re-runs back-fill missing files but never overwrite existing ones.
- `/wiki/ideation/README.md` and `/wiki/ideation/.gitkeep` — written from `references/wiki-templates/ideation-readme.md` and an empty file.
- `/wiki/index.md` is now sourced from `references/wiki-index-template.md` (extracted from inline SKILL.md content) — the bundled template covers all three layers with the standard category headings.

**Read-side propagation (Step 3):**

The skill now reads seven references at the start of each run: `claude-template.md`, `persona.md`, `memory-protocol.md`, `wiki-index-template.md`, plus the four generative templates under `references/wiki-templates/`. Missing any of them is treated as a broken plugin install and aborts.

**Log + confirmation updates (Step 6 + Step 7):**

Both the `init` and the `template-sync` log entries gained a generative-layer line listing the per-file state (`created` / `already present`). The Step 7 confirmation surfaces the same.

## Discipline preserved

The CB-005 changes are **additive** — every existing contract still holds:

- `memory-protocol.md` is still propagated via additive merge (Step 6); the upgrade to the three-layer protocol reaches existing projects through re-runs.
- `index.md` is **not** additively merged; if a project predates the generative-layer headings, the user's hand-curated catalog is preserved and the new headings are added by hand (the user-curated catalog and the bundled headings interleave too freely for safe additive merge).
- `CLAUDE.md` re-run merge is unchanged — additive, except for the deprecated-sections channel (Step 5.5).
- Persona block is preserved across re-runs (use `/yuval-core:refresh-persona` to update).
- All new generative-layer files are create-if-missing — never overwritten.

## Re-run on a pre-D-029 project

Re-running `init` on a project that predates the generative layer back-fills the missing files:

- `/wiki/vision.md`, `goals.md`, `spirit-signals.md`, `backlog.md`, `ideation/README.md`, `ideation/.gitkeep` → all `created`.
- `/wiki/memory-protocol.md` → `refreshed` (additive merge picks up the three-layer protocol updates).
- `/wiki/index.md` → `already present` (left untouched; user adds the generative-layer headings by hand if desired).

The net effect is that an existing project gains the workshop layer without losing any of its existing wiki state.

## Validation

The new scaffolding is verifiable by running `init` against a scratch project:

```bash
mkdir /tmp/init-cb005-scratch && cd /tmp/init-cb005-scratch
# Run /yuval-core:init in this folder
ls -la wiki/                   # should show vision.md, goals.md, spirit-signals.md, backlog.md, ideation/
ls wiki/ideation/              # should show README.md and .gitkeep
grep -A2 "Generative layer" wiki/index.md      # should show the new section heading
grep -c "Generative" wiki/memory-protocol.md   # should match (D-029 references in the protocol)
```

Re-running `init` on this workbench's parent folder is also valid: every block is create-if-missing, the merge for `CLAUDE.md` is strictly additive, and the workbench's hand-migrated generative files (vision/goals/spirit-signals/backlog/ideation) are left untouched.

## References

- Bundled SKILL: [`plugins/yuval-core/skills/init/SKILL.md`](../../plugins/yuval-core/skills/init/SKILL.md)
- Bundled references: [`plugins/yuval-core/skills/init/references/`](../../plugins/yuval-core/skills/init/references/) — `claude-template.md`, `persona.md`, `memory-protocol.md`, `wiki-index-template.md`, `wiki-templates/`, `deprecated-sections.md`.
- ADRs: [D-016](../../DECISIONS.md#d-016) (each skill carries its own `references/`), [D-018](../../DECISIONS.md#d-018) (memory protocol lives in the wiki), [D-029](../../DECISIONS.md#d-029) (Wiki 2.0: workshop layer).
- Concept: [`concept-wiki-workshop.md`](concept-wiki-workshop.md) — workshop stance + three-layer model + rejected SoT alternative.
- Brief: [`CB-005`](../../../output/briefs/CB-005-wiki-workshop-layer.md) (frozen post-promotion).
