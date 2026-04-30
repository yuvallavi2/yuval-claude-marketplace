# Next — claude-marketplace-dev (code)

_Last updated: 2026-04-30_

## Next action

Push the local commits and tags to GitHub: `git push origin main && git push origin brief/CB-003 v0.8.0` (and any unpushed earlier tags: `brief/CB-001`, `v0.6.0`, `brief/CB-002`, `v0.7.0`). Then schedule the deferred end-to-end runtime validation pass for CB-003 in a fresh scratch project — confirm the byte-correct in-place replacement on a marker-bearing CLAUDE.md, and confirm the abort path fires on a marker-less one.

## Why

CB-003 just landed: refresh-persona skill shipped, plugin bumped to v0.8.0, D-027 recorded as a sub-decision under D-026. brief/CB-003 + v0.8.0 tags created locally. Structural validation passed (files in the right place, references resolve, plugin manifest updated). Behavioral validation in a scratch project is the only acceptance gap.

## Open threads

- Behavioral validation deferred for CB-001 (§10 tests 1–7), CB-002 (proactive triggers fire at the right moments), and now CB-003 (refresh-persona end-to-end). Worth bundling into a single dedicated "validation pass" session.
- plugin.json was stale at 0.6.0 through CB-002 (marketplace.json had been bumped to 0.7.0 alone); CB-003 brings both to 0.8.0. Worth a follow-up: every brief that bumps the version should touch both files. Could be enforced by a small linter or a checklist line in the close-brief routine.
- Bundled `persona.md` in refresh-persona's references/ retains the original "(future)" wording in its HTML comment ("refreshed by /refresh-persona (future)"). Slightly stale now that the skill exists. Defer to whichever brief next touches persona content.
- Several unpushed tags accumulating locally (brief/CB-001, v0.6.0, brief/CB-002, v0.7.0, brief/CB-003, v0.8.0). Push them all in one go when the user is ready.

## Recent context

- Last session: implementation of CB-003 (this session). Plugin: v0.6.0 plugin.json / v0.7.0 marketplace.json → v0.8.0 both. ADRs: D-027 added.
