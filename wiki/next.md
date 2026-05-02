# Next — claude-marketplace-dev (code)

_Last updated: 2026-05-02_

## Next action

Push v0.8.1: `git push origin main && git push origin v0.8.1`. Then run the deferred bundled behavioral-validation pass for CB-001 / CB-002 / CB-003 in a single fresh-scratch-project session — and verify the persona v2 lands cleanly on a fresh `init` (mode-switcher diagram renders, banned-phrases list intact).

## Why

Persona v2 just landed (v0.8.1 release) — mode-switcher (brainstorm/plan/implement), visual-first rule, disagreement protocol, expanded banned-phrases. Workbench dogfooded: refresh-persona ran in place against `/Users/ylavi/claude-marketplace-dev/CLAUDE.md`, replaced the persona block byte-correct between markers. First real-world exercise of the refresh skill — passed. Behavioral validation in a fresh init is the remaining acceptance gap (along with the deferred CB-001/CB-002 validations).

## Open threads

- Validation pass still bundled and deferred (CB-001 §10 tests, CB-002 proactive triggers, CB-003 refresh-persona, now also persona v2 first-init).
- Add a close-brief / close-release checklist line: bump both `plugin.json` and `marketplace.json` together. CB-003 had to fix the stale plugin.json from CB-002.
- Bundled `persona.md` in both copies retains "(future)" wording in the HTML comment preamble. Cosmetic; update next time persona content is touched.

## Recent context

- 2026-05-02: persona v2 shipped as v0.8.1; workbench CLAUDE.md persona block refreshed in place. Local tag v0.8.1 created; push pending.
- 2026-04-30: CB-003 implemented (refresh-persona skill); pushed to origin same day.
