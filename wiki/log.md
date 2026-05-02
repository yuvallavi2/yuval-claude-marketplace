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

## [2026-04-27] close-brief | CB-002 closed at tag brief/CB-002
- Brief: ../../output/briefs/CB-002-proactive-wiki-protocol.md (status: promoted → closed)
- Tag created (local): brief/CB-002 (annotated)
- Release tag created (local): v0.7.0
- Push: pending user instruction
- Scope met: Proactive triggers section added to memory-protocol.md (table + Live session page + Surfacing convention); claude-template.md Rules block points at the new section; init SKILL.md index.md template has the Briefs category; workbench migrated (memory-protocol.md refreshed, index.md Briefs entries updated for CB-001 closure and CB-002 promotion); plugin bumped 0.6.0 → 0.7.0
- Scope deferred: runtime validation in a fresh scratch project (acceptance criterion 5) — structural validation of the references passed; runtime validation requires Yuval to mount a fresh project and run init

## [2026-04-27] release | v0.7.0 (local tag)
- Plugin: yuval-core 0.6.0 → 0.7.0
- Includes brief: brief/CB-002
- Tag: annotated, local — push pending user instruction
- Description: proactive wiki protocol — adds Proactive triggers section + Live session page convention + Surfacing convention to memory-protocol.md; adds Briefs category to init's index.md template; claude-template.md Rules block points at the new triggers section

## [2026-04-30] promote | CB-003 → D-026
- Brief: ../../output/briefs/CB-003-implement-refresh-persona.md
- ADR: D-026 in DECISIONS.md
- Status: ready for implementation

## [2026-04-30] adr | D-027 — refresh-persona implementation: skill shape & missing-markers stance
- Sub-decision under D-026, recorded mid-implementation per the brief's promote step
- (a) Implemented as a skill (skills/refresh-persona/) per D-016, not a flat command file
- (b) Missing markers → abort with guidance, not auto-insert (collision-risk; matches D-018's manual-cleanup precedent)

## [2026-04-30] implement | CB-003 — refresh-persona skill shipped
- Created: plugins/yuval-core/skills/refresh-persona/SKILL.md, plugins/yuval-core/skills/refresh-persona/references/persona.md (verbatim copy of init's persona.md at ship time)
- Updated: plugins/yuval-core/.claude-plugin/plugin.json (0.6.0 → 0.8.0; refresh-persona added to keywords; description refreshed) — note plugin.json had been left at 0.6.0 during CB-002, marketplace.json was at 0.7.0; both now realigned at 0.8.0
- Updated: .claude-plugin/marketplace.json (0.7.0 → 0.8.0)
- Updated: plugins/yuval-core/README.md (added refresh-persona to skills list; removed from Planned; refreshed persona-refresh principle line)
- Updated: code/DECISIONS.md (D-027 added)
- Updated: code/wiki/index.md (D-025, D-026, D-027 entries added under Decisions)

## [2026-04-30] close-brief | CB-003 closed at tag brief/CB-003
- Brief: ../../output/briefs/CB-003-implement-refresh-persona.md (status: promoted → closed)
- Tag created (local): brief/CB-003 (annotated)
- Release tag created (local): v0.8.0
- Push: pending user instruction
- Scope met: refresh-persona skill (SKILL.md + bundled persona.md), plugin v0.8.0 bump, README updated, workbench wiki migrated (Skills entry + design page), DECISIONS.md D-027 added
- Scope deferred: end-to-end runtime validation in a scratch project (acceptance criteria 3–4) — structural validation passed; behavioral validation requires Yuval to mount a fresh project with the new plugin version

## [2026-04-30] release | v0.8.0 (local tag)
- Plugin: yuval-core 0.6.0 → 0.8.0 (plugin.json was stale at 0.6.0; marketplace.json was at 0.7.0; both now realigned at 0.8.0)
- Includes brief: brief/CB-003
- Tag: annotated, local — push pending user instruction
- Description: ships /yuval-core:refresh-persona skill — re-syncs CLAUDE.md persona block in already-initialized projects without re-running init; aborts with guidance on missing markers (no auto-insert)

## [2026-05-02] push | CB-003 commit and tags pushed to origin
- Pushed: main (d65d881), tags brief/CB-003 and v0.8.0
- origin/main now in sync; CB-003 acceptance criterion 7 (annotated tag pushed) satisfied
- Earlier tags (brief/CB-001, v0.6.0, brief/CB-002, v0.7.0) were already on the remote; the prior next.md hand-off had overstated the unpushed list

## [2026-05-02] release | v0.8.1 — persona v2 (mode-switcher + visual-first)
- Persona content edit, no brief, no ADR (per D-023 — content edits don't warrant briefs)
- Updated: skills/init/references/persona.md, skills/refresh-persona/references/persona.md (verbatim copies, both)
- New persona structure: Who I am / How I read / Mode-switcher / Disagreement protocol / Tone / Language
- Big additions: 3-mode switcher (brainstorm/plan/implement) with trigger phrases per mode; visual-first rule (mermaid/ASCII/SVG defaults); disagreement protocol with 5 explicit don'ts; banned-phrase list extended (deep dive, going forward, etc.)
- Cuts: generic "direct & concise", "challenge don't flatter", "ask before assuming", "one question at a time" — folded into more concrete rules
- Plugin: 0.8.0 → 0.8.1 (both plugin.json and marketplace.json)
- Dogfood: workbench's own /Users/ylavi/claude-marketplace-dev/CLAUDE.md persona block refreshed in place (markers preserved, surrounding content byte-identical) — first real-world exercise of /yuval-core:refresh-persona
- Tag: v0.8.1 (annotated)
