# Code Wiki Log — claude-marketplace-dev

Append-only chronological record of code-side operations: promotions received, sessions, ADRs, report-back events, lints, releases. Each entry is prefixed with `## [YYYY-MM-DD] operation | title` so the log is grep-friendly:

```bash
grep "^## \[" wiki/log.md | tail -10
grep "^## \[.*promote" wiki/log.md
grep "^## \[.*adr" wiki/log.md
```

**Operations vocabulary:**
- **promote** — new brief arrived from ideation (auto-logged by `promote-to-code`)
- **implement** — a promoted brief was implemented (manual entry at session end)
- **report-back** — finding pushed to ideation (auto-logged by `report-back`)
- **session** — generic code-side work session (manual entry)
- **lint** — code-wiki health check
- **adr** — new D-xxx recorded outside a promotion (refactor decisions, etc.)
- **close-brief** — brief closed; annotated tag created
- **release** — version tag created

**Rules:** Append only. Never edit or delete past entries. Each entry is a few bullets, not prose.

---

## [2026-04-27] init-code | Code wiki scaffolded (workbench migration)
- /code/ already existed as a git repo (cloned from yuvallavi2/yuval-claude-marketplace)
- `init-code` skill applied in re-run-existing-repo mode: skipped `git init`, additively scaffolded missing pieces only
- Created: /code/CLAUDE.md, /code/wiki/memory-protocol.md, /code/wiki/index.md, /code/wiki/log.md, /code/wiki/next.md
- Left alone: /code/DECISIONS.md (existing, append-only), /code/.gitignore (existing), top-level README intentionally omitted (plugin README serves that role at plugins/yuval-core/README.md)
- Status: Code wiki ready; this entry is the bootstrap for the back-fill below.

## [2026-04-27] promote | CB-001 → D-019 (back-filled)
- Brief: ../../output/briefs/CB-001-bootstrap-code-workspace.md
- ADR: D-019 in DECISIONS.md (with structural sub-decisions D-020 through D-024 capturing the design's individual commitments)
- Status: in flight — implementation underway in this session
- Bootstrap caveat: hand-authored. `/yuval-core:promote-to-code` is itself part of CB-001's scope; ADR was added manually, brief was frozen manually, both wiki logs got `promote` entries manually. This entry was back-filled after `init-code` semantics created `/code/wiki/log.md`. The hand-authored steps are the validation that the command's contract is correct — once `promote-to-code` exists, future briefs flow through it automatically.

## [2026-04-27] close-brief | CB-001 closed at tag brief/CB-001
- Brief: ../../output/briefs/CB-001-bootstrap-code-workspace.md (status: promoted → closed)
- Tag created (local): brief/CB-001 (annotated)
- Release tag created (local): v0.6.0
- Push: pending user instruction
- Scope met: init-code, write-brief, promote-to-code, report-back, init modifications (briefs scaffold + Code workspace pointer + extended deprecated-sections), D-019 through D-024, plugin bumped to v0.6.0, workbench migration applied
- Scope deferred: validation in a fresh scratch project per design §10 tests 1–7; push to GitHub main; tag push

## [2026-04-27] release | v0.6.0 (local tag)
- Plugin: yuval-core 0.5.0 → 0.6.0
- Includes brief: brief/CB-001
- Tag: annotated, local — push pending user instruction
- Description: code-workspace extension (init-code, write-brief, promote-to-code, report-back) + briefs scaffolding + extended deprecated-sections channel for legacy memory-protocol migration

## [2026-04-27] promote | CB-002 → D-025
- Brief: ../../output/briefs/CB-002-proactive-wiki-protocol.md
- ADR: D-025 in DECISIONS.md
- Status: ready for implementation
