# Next — claude-marketplace-dev (code)

_Last updated: 2026-05-04_

## Next action

**User: say "commit and push" to land CB-005 closure.** I'll then: (a) flip CB-005 status field to `closed` and append the closing status-log entry, (b) make the implementation commit, (c) create annotated tags `brief/CB-005` and `v0.10.0`, (d) push (bundle with the still-pending pushes from CB-004 / v0.9.0 and v0.8.1 carryover).

After CB-005 lands: return to ideation, run `/yuval-core:promote-to-code CB-006` (D-030 already added), then implement CB-006 (wiki-aware skills — wires `write-brief`, `refresh-spirit`, and `promote-to-code` to consume the workshop layer this brief just shipped).

## Why

CB-005 was implemented in this session (Claude Code). Plugin v0.10.0 build is complete: all 5 wiki templates bundled, wiki-index-template.md extracted + extended with three-layer headings, memory-protocol.md upgraded to the workshop model with new triggers + extended session-start read sequence, SKILL.md scaffolds the new generative layer, README.md documents the contract, plugin.json + marketplace.json bumped in lockstep. Workbench `/wiki/` hand-migrated (vision was already authored last session; goals / spirit-signals (n/a) / backlog / ideation/ created; memory-protocol overwritten; index restructured to three layers preserving workbench-specific override categories). Code-wiki gains `concept-wiki-workshop.md` + `skill-init.md`. Validated structurally against a /tmp scratch project.

Acceptance criteria met:
- AC1 (init scaffolds new files from bundled templates) ✓
- AC2 (memory-protocol.md is three-layer + new triggers + new read sequence) ✓
- AC3 (wiki-index-template.md includes generative-layer headings) ✓
- AC4 (workbench /wiki/ migrated) ✓
- AC5 (D-029 in DECISIONS.md — added at promotion time) ✓
- AC6 (README.md documents three-layer model) ✓
- AC7 (concept-wiki-workshop.md exists) ✓
- AC8 (skill-init.md exists) ✓
- AC9 (scratch-project validation) ✓ structural; behavioral validation deferred to bundled fresh-Claude pass (same gap as CB-001/2/3/4)
- AC10 (annotated tags brief/CB-005 + v0.10.0) — pending closure commit

## Open threads

- **Closure commit + tags pending.** User "commit and push" lands CB-005 + applies tags + pushes (along with CB-004's still-unpushed v0.9.0 + brief/CB-004 + carryover v0.8.1).
- **CB-006 implementation queued.** Brief at `/output/briefs/CB-006-wiki-aware-skills.md`. D-030 already added at promotion. Run `/yuval-core:promote-to-code CB-006` from ideation, then implement.
- **Behavioral-validation pass still deferred.** Now bundles CB-002 proactive triggers + persona v2 mode-switcher + CB-004 SPIRIT walk + CB-005 init scaffolding (manual verification: run `init` in a fresh project, confirm all generative files appear with correct templates) + eventually CB-006 skill consumption. Run in a fresh Claude session, not same-Claude-just-authored-this.
- **F1, F2 housekeeping fixes** still pending — bundle into a v0.10.x patch when the housekeeping mood strikes (init-code's empty `code-wiki-templates/` dir; write-brief's undocumented placeholder names).

## Recent context

- 2026-05-04 (this session, code): CB-005 implemented end-to-end. Plugin v0.10.0 build complete; workbench `/wiki/` hand-migrated; concept-wiki-workshop.md + skill-init.md authored; structural validation passed. Closure commit + tags pending user instruction.
- 2026-05-04 (prior, ideation): CB-005 + CB-006 authored, both promoted (D-029, D-030); workbench `/wiki/vision.md` hand-migrated.
- 2026-05-03: CB-004 implemented end-to-end (v0.9.0); push still pending.
- 2026-05-02: persona v2 (v0.8.1); structural validation pass; F1/F2 logged.
