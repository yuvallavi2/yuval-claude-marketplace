# Next — claude-marketplace-dev (code)

_Last updated: 2026-05-02_

## Next action

Two threads, in order of weight:

1. **Behavioral validation** (genuinely needs a fresh Claude session). Open a new Claude Code session in `/tmp/persona-v2-scratch` (or any fresh folder), send brainstorm-mode / plan-mode / implement-mode prompts, verify the persona v2 mode-switcher actually shifts Claude's behavior. Same for CB-002 proactive triggers — do they fire at the right moments cold, without being primed?
2. **Patch the two findings** (F1: init-code's empty `code-wiki-templates/` dir; F2: undocumented placeholder names in write-brief's brief-template.md). Small, can land in a single housekeeping patch — call it v0.8.2.

## Why

Bundled structural validation just ran in `/tmp/persona-v2-scratch` and passed all 6 tests (init / init-code / write-brief / promote-to-code / report-back / refresh-persona both paths). Plugin-manifest sync rule codified in `/code/CLAUDE.md`. Framework-extraction-spec marked superseded (banner pointing at D-018). Two real bugs surfaced (F1, F2) — captured in the log as `adr` entries pending fix decision. Behavioral validation is the only remaining gap, and only because same-Claude-just-authored-this can't validate itself cold.

## Open threads

- F1, F2 fixes pending — bundle into v0.8.2 patch when the housekeeping mood strikes.
- Behavioral validation still pending (intentionally — fresh-session-required).

## Recent context

- 2026-05-02: persona v2 shipped (v0.8.1); structural validation passed; sync-rule codified; framework-extraction-spec marked superseded; F1/F2 logged.
- 2026-04-30: CB-003 implemented (refresh-persona skill); pushed to origin same day.
