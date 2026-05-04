# Next — claude-marketplace-dev (code)

_Last updated: 2026-05-04_

## Next action

**User: say "commit and push" to land CB-006 closure.** I'll then make the implementation commit, create annotated tags `brief/CB-006` and `v0.11.0`, and push. CB-006's brief status field is already flipped to `closed` and the closing status-log entry is in place.

After CB-006 lands, the wiki-aware MVP is shipped. Remaining work is non-blocking: bundled behavioral-validation pass (deferred since CB-001), F1/F2 housekeeping fixes (deferred since 2026-05-02), and any new briefs the user authors.

## Why

CB-006 was implemented in this session (Claude Code, same date as CB-005). Plugin v0.11.0 build is complete:
- `memory-protocol.md` gains the "Skill-wiki contract (D-030)" subsection.
- `write-brief` reads `/wiki/goals.md` (Step 1.7) + `/wiki/backlog.md` (Step 1.8, LLM-judged matching) + `/wiki/ideation/*.md` (Step 3.5 References); stashes `<!-- backlog-closes: ... -->` and `<!-- ideation-promotes: ... -->` markers in the brief.
- `refresh-spirit` reads `/wiki/spirit-signals.md` pending entries (Step 3.5), folds them into pre-fills, marks them synthesized after SPIRIT.md write (Step 7.5, append-only).
- `promote-to-code` parses brief markers and (only when present) closes matching backlog items (Step 5.5) + archives consumed ideation files (Step 5.6).
- `README.md` documents the three-rule contract + per-skill table + brief-marker convention.
- `plugin.json` + `marketplace.json` bumped to v0.11.0 in lockstep.
- `D-030` expanded from stub to full enumeration of (a)-(e) per AC4.
- Code-wiki gains `skill-write-brief.md`, `command-promote-to-code.md`, `concept-skill-wiki-contract.md`; `skill-refresh-spirit.md` gets a CB-006-evolution section appended.
- Workbench `/wiki/goals.md` and `/wiki/index.md` (Briefs section) updated to reflect CB-005 + CB-006 closure.

Acceptance criteria met:
- AC1 (write-brief reads goals + backlog + ideation; sparse-wiki fallback) ✓
- AC2 (refresh-spirit reads pending signals + marks synthesized; append-only) ✓
- AC3 (promote-to-code parses markers + closes backlog + archives ideation) ✓
- AC4 (D-030 captures the five points) ✓
- AC5 (CB-005 templates already use the formats this brief depends on — verified during CB-005 implementation) ✓
- AC6 (memory-protocol.md documents the contract) ✓
- AC7 (README.md documents contract + per-skill surfaces + markers) ✓
- AC8 (wiki pages: skill-write-brief, skill-refresh-spirit CB-006 section, command-promote-to-code, concept-skill-wiki-contract) ✓
- AC9 (workbench validation: signal → synthesis, backlog → close, ideation → archive) — deferred to bundled behavioral pass (cannot self-validate)
- AC10 (scratch-project sparse-wiki path validation) — deferred to same pass
- AC11 (annotated tags brief/CB-006 + v0.11.0) — pending closure commit

## Open threads

- **Closure commit + tags pending.** User "commit and push" lands CB-006 + applies tags + pushes.
- **Bundled behavioral-validation pass** still pending — now covers CB-002 / persona v2 / CB-004 (SPIRIT walk + refresh-spirit + promote-to-code refusal) / CB-005 (init scaffolding new files) / CB-006 (write-brief reads + refresh-spirit signals fold + promote-to-code marker side-effects). Run in a fresh Claude session, not same-Claude-just-authored-this. The single biggest deferred validation surface in the project right now.
- **F1, F2 housekeeping fixes from 2026-05-02** still pending — bundle into a v0.11.x patch when the housekeeping mood strikes (init-code's empty `code-wiki-templates/` dir; write-brief's undocumented placeholder names).
- **Drift policing for `spirit-prompts.md`** (two copies: write-brief, refresh-spirit) — D-016 accepts; revisit only if drift becomes a real problem.

## Recent context

- 2026-05-04 (this session, code): CB-006 implemented end-to-end. Plugin v0.11.0 build complete. Three wiki-aware skills shipped (write-brief / refresh-spirit / promote-to-code); D-030 expanded; four code-wiki pages authored or extended; workbench goals + briefs index updated. Closure commit + tags pending.
- 2026-05-04 (earlier today, code): CB-005 implemented end-to-end (v0.10.0); pushed to origin/main with tags `brief/CB-005` + `v0.10.0`.
- 2026-05-04 (prior, ideation): CB-005 + CB-006 authored, both promoted (D-029, D-030); workbench `/wiki/vision.md` hand-migrated.
- 2026-05-03: CB-004 implemented (v0.9.0).
- 2026-05-02: persona v2 (v0.8.1); F1/F2 logged.
