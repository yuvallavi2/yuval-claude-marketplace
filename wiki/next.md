# Next — claude-marketplace-dev (code)

_Last updated: 2026-04-27_

## Next action

Run validation per `/output/code-workspace-design.md` §10 in a fresh scratch project: ideation init, `init-code` first run, `init-code` re-run, `write-brief`, `promote-to-code CB-001`, `report-back`, brief closure with annotated tag.

Once validation passes, close `CB-001`: update brief status → `closed`, append closure note to status log, create annotated `brief/CB-001` tag at HEAD, append `close-brief` entries to both wiki logs, push.

## Why

CB-001's implementation just landed: `init-code`, `write-brief`, `promote-to-code`, `report-back`, modifications to `init`, D-019…D-024 in DECISIONS.md, plugin bumped to v0.6.0, workbench migrated. Validation in a scratch project is the last acceptance criterion before closure.

## Open threads

- The framework-extraction-spec at `../../output/framework-extraction-spec.md` is partially superseded by D-019. Decide post-CB-001 whether to retire, mark deprecated, or keep as historical record.
- CB-002 (proactive wiki protocol) was authored mid-session in `/output/briefs/`. It is open and depends on CB-001 closure — pick up once CB-001 is closed.
- Validation in a scratch project hasn't run yet. Recommended order: a brand-new empty folder, run `/yuval-core:init`, walk through the §10 tests one by one.

## Recent context

- Last session: implementation of CB-001 (this session). Plugin version: v0.6.0. ADRs: D-019 through D-024 added.
