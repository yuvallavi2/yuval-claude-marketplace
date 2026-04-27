# Marketplace Design Decisions

A running log of structural decisions, with the reasoning. Read this before making architectural changes — most "improvements" turn out to be re-opening a decision that was closed for a reason.

Each entry has three parts: **Decision** (what), **Reasoning** (why), and **Rejected alternatives** (what we considered and didn't pick).

---

## D-001 — Two wikis, not one

**Decision:** Each project has two separate wikis: a product/strategy wiki at the project root (`wiki/`), and an engineering wiki inside the code repo (`code/wiki/`). They never merge.

**Reasoning:** The two wikis have genuinely different entities. The product wiki tracks decisions, stakeholders, open questions, PRDs. The engineering wiki tracks ADRs, API contracts, components, runbooks. Forcing them into one wiki creates schema drift and audience mismatch. Separate wikis also respect git reality — the code wiki ships with the code repo; the product wiki stays outside it.

**Rejected alternatives:**
- *Single wiki covering everything* — mixes audiences, creates schema drift, leaks strategy into code repos on handoff.
- *Shared wiki via symlink or submodule* — elegant in theory, operationally fragile (Windows symlink issues, submodule annoyances, permission complications). Collapses under real use.

---

## D-002 — Unified project folder, not split thinking/code folders

**Decision:** One project folder per initiative. Contains `input/`, `in-progress/`, `output/`, `wiki/` (product), and — only when needed — `code/` as a subfolder.

**Reasoning:** Solo operator. Coordination cost of maintaining two separate top-level folders (one for thinking, one for code) is paid per project, forever, with no compensating benefit. One roof keeps related work physically co-located.

**Rejected alternatives:**
- *Two separate top-level folders linked by reference* — cleanest conceptual separation but doubles setup cost and forces cross-folder navigation for related work.

---

## D-003 — `code/` is optional, created by a separate command

**Decision:** `init` does NOT create `code/`. A separate future command (`init-code`) adds the code realm when a project reaches implementation.

**Reasoning:** Not every project becomes code. Many end as a PowerPoint, Excel, or other artifact. Creating an empty `code/` in every project adds clutter and implies commitment to code work that may never happen.

**Rejected alternatives:**
- *Always create `code/` in init* — pollutes thinking-only projects with unused git repo scaffolding.

---

## D-004 — Parent folder is not git, `code/` is its own repo

**Decision:** The project root (`my-project/`) is a plain unversioned folder. When `code/` exists, it is an independent git repo. The parent is never a repo.

**Reasoning:** Thinking artifacts (PowerPoints, PDFs, Excel) don't benefit from git. Code needs git for deploy pipelines, history, collaboration. Making the parent a repo forces git overhead on non-code work and creates awkward submodule situations.

**Rejected alternatives:**
- *Parent is git, `code/` is a submodule* — submodule complexity for no real benefit.
- *Everything in one mega-repo* — strategy docs and code in same repo means strategy leaks when code is shared or open-sourced.

---

## D-005 — Output → code handoff is "promote and freeze"

**Decision:** When a document graduates from `output/` to the code realm, it is *copied* into `code/wiki/` as an immutable, date-stamped reference (ADR-style). The original `output/` version becomes historical; the `code/wiki/` version becomes engineering's truth.

**Reasoning:** Clean contract with clear ownership. No ambiguity about which version is current in which realm. No shared-state drift. `code/wiki/` can evolve as implementation teaches new things without mutating the original product decision record.

**Rejected alternatives:**
- *Reference without copy* — `code/` reads from `../output/`. Couples the two realms, breaks code repo portability.
- *Copy and allow both to evolve* — creates drift without clear source of truth.

---

## D-006 — One persona, Cowork-scoped (initially)

**Decision:** Single persona, currently scoped to Cowork (injected into each project's `CLAUDE.md` by `init`). Not yet extended to Claude Code or other surfaces.

**Reasoning:** Gravity of work is in Cowork. Adding persona to Claude Code can wait until there's real friction from not having it.

**Rejected alternatives:**
- *Multiple persona modes (brainstorm vs coding vs writing)* — premature without evidence that a single persona fails.
- *Global persona across all surfaces* — Claude.ai has no file-read capability; "global" has to be different things on different surfaces. Deferred.

---

## D-007 — Persona lives in a separate file, refreshable decoupled from init

**Decision:** Persona content lives in `templates/persona.md`, not inside the `claude-template.md` directly. `init` merges persona into the template at project creation. A future `/refresh-persona` command replaces the persona block in an already-initialized project's `CLAUDE.md`.

**Reasoning:** Persona evolves independently of template structure. Coupling them means every persona edit requires re-running init, which is invasive. Decoupling lets persona be a living document that can be refreshed into existing projects without disturbing other work.

**Implementation:** Persona block in `CLAUDE.md` is wrapped in `<!-- PERSONA:START -->` and `<!-- PERSONA:END -->` markers. `init` replaces content between markers on first run; skips on re-run. `/refresh-persona` (future) replaces content between markers any time.

**Rejected alternatives:**
- *Persona inline in template* — couples persona updates to template updates, creates merge complexity.
- *Automatic persona refresh on every init re-run* — silently overwrites any project-local persona customization the user may have made.

---

## D-008 — Single marketplace serves both Cowork and Claude Code

**Decision:** One marketplace folder/repo, consumed identically by both surfaces via `/plugin marketplace add`. No surface-specific distribution mechanism.

**Reasoning:** Both surfaces support the same plugin/marketplace architecture. Maintaining two parallel distribution paths (e.g., symlinks for Cowork, marketplace for Claude Code) means twice the work and a high risk of versions drifting.

**Rejected alternatives:**
- *Symlinks for Cowork, marketplace for Claude Code* — two mechanisms, guaranteed drift.
- *Copy skills into each project's `CLAUDE.md` via init* — works, but any skill update requires re-running init on every project.

---

## D-009 — Marketplace is a GitHub repo (skipped local-folder phase)

**Decision:** Marketplace lives as a public GitHub repo (`yuval-claude-marketplace`), installed via `/plugin marketplace add <handle>/yuval-claude-marketplace`.

**Reasoning:** Local-folder marketplace install didn't work reliably in our environment. GitHub was the intended destination anyway. Going straight there removes an intermediate phase and its maintenance overhead.

**Rejected alternatives:**
- *Local folder only, GitHub later* — original plan, abandoned when local install failed.
- *Private repo* — considered; kept public after sanitizing persona to remove employer mention. Public makes install simpler (no auth) and matches the open-source norm for tools like this.

---

## D-010 — `llm-wiki` is ONE skill bundling ingest/query/lint

**Decision:** Wiki operations (`ingest`, `query`, `lint`) are bundled in a single `llm-wiki` skill rather than three separate skills.

**Reasoning:** The three operations share data structures (index.md format, log.md format, page template) and schema assumptions. Separating them forces duplication of the schema across skills and creates version-skew risk. Bundled skill owns the schema end-to-end.

**Rejected alternatives:**
- *Three separate skills* — modular in theory, but the operations aren't independently useful; you always want all three.

---

## D-011 — Template teaches wiki behavior ambiently

**Decision:** The `claude-template.md` written to each project includes explicit wiki discipline — ownership rules (LLM owns wiki, user owns input/in-progress/output), operation vocabulary (ingest, query, lint), and structural conventions (index.md, log.md, pages/).

**Reasoning:** Per Karpathy: the schema document is "what makes the LLM a disciplined wiki maintainer rather than a generic chatbot." Without it, folders alone don't produce wiki behavior. Teaching behavior in the template means `init` alone yields a working wiki-aware project, even before the `llm-wiki` skill is installed.

**Rejected alternatives:**
- *Template stays skeletal, all wiki behavior comes from llm-wiki skill* — cleaner separation of concerns, but means init-only projects have dead folders with no behavior attached.

---

## D-012 — `wiki/log.md` replaces `in-progress/session-log.md`

**Decision:** Session logs and wiki operation logs both go in `wiki/log.md` — a single append-only chronological record. No separate session log in `in-progress/`.

**Reasoning:** `in-progress/` is the "deletable scratch" zone. Anything meant to persist doesn't belong there. Session log is persistent historical record, so it belongs in `wiki/`. Karpathy's `log.md` format (`## [YYYY-MM-DD] operation | title`) is grep-friendly and already the right shape for session entries.

**Rejected alternatives:**
- *Two logs (session log in in-progress, wiki log in wiki)* — two places to look, two formats to maintain, no clear division.
- *Session log only, no wiki log* — loses Karpathy's structured log format and its grep-friendliness.

---

## D-013 — Templates read at runtime from `~/claude-marketplace/templates/`, not from plugin cache

**Decision:** `init` reads `claude-template.md` and `persona.md` from a fixed path on the user's machine (`~/claude-marketplace/templates/`), not from the installed plugin bundle.

**Reasoning:** Plugin bundles are effectively immutable — they're overwritten on update. If templates live only inside the bundle, editing them requires either editing the GitHub repo (slow) or editing the cache (gets overwritten). A known editable location at `~/claude-marketplace/templates/` lets the user edit once and have every future `init` pick it up, without any reinstall.

**Tradeoff accepted:** Setup on each machine requires copying templates from the installed plugin into `~/claude-marketplace/templates/` once. Documented in the repo README. Worth the tradeoff for the "edit once, propagates" workflow.

**Rejected alternatives:**
- *Read templates from plugin bundle only* — simpler install, but edits require either going through GitHub or editing an overwrite-prone cache.

---

## D-014 — Commit workflow: Claude-assisted in Claude Code

**Decision:** Changes to the marketplace repo are committed via Claude Code — user edits files, asks Claude to commit and push with a sensible message. User reviews diff and message before push.

**Reasoning:** Speed without the footgun of auto-commit. Humans-in-the-loop catches accidents (half-finished edits, unintended persona leaks). No new command to maintain.

**Rejected alternatives:**
- *Fully manual (user types git commands)* — works, but Claude is already in the loop so might as well use it.
- *Auto-commit via a `/publish-marketplace` command* — too easy to ship unfinished work; revisit if manual commits become tedious.

---

## D-015 — `main` branch only, no feature branches

**Decision:** Solo development on `main`. No branch protection, no feature branches, no PR workflow.

**Reasoning:** Solo user. Branching is overhead without a reason. Discipline is to test changes in a scratch project before pushing.

**Revisit:** When more than one person contributes, or when a broken push has impacted real work.

---

## D-016 — Templates live inside each skill's `references/` folder, not in a shared location (supersedes D-013)

**Decision:** Templates and persona files are bundled inside each skill's own `references/` folder. The top-level `templates/` folder and the external `~/claude-marketplace/templates/` path are eliminated. Each skill that needs a template or persona carries its own copy. Version control and propagation happen through the git repo and plugin updates.

**Reasoning:** D-013 introduced a two-location system (repo `templates/` + user-local `~/claude-marketplace/templates/`) to allow edits without reinstalling. In practice this created an unsolved sync problem across machines and a manual setup step that added friction without real benefit. The git repo already handles versioning and distribution — adding a second editable location on top of it was redundant. Keeping templates inside each skill's `references/` also preserves skill independence: no skill reaches into a sibling's folder or depends on an external path.

**What changed:**
- `templates/` folder deleted from repo root
- `persona.md` moved into `skills/init/references/`
- `init` SKILL.md updated to read from `references/` as the primary path
- Any future skill needing persona (e.g., `refresh-persona`) carries its own copy in its `references/`

**Rejected alternatives:**
- *Shared `references/` at the plugin level* — avoids duplication but creates cross-skill coupling. Skills should be self-contained.
- *Keep D-013 as-is* — the external path was an unsolved problem (see "Template sync on multiple machines" open question, now closed).

---

## D-017 — Session continuity via a dedicated `/wiki/next.md` hand-off file

**Decision:** Session-to-session continuity is carried by a single file, `/wiki/next.md`, alongside the append-only `/wiki/log.md`. Every session that produces meaningful work overwrites `next.md` with the current hand-off (next action, why, open threads, recent context) and prints its contents as the closing chat message — the "hot swap." The next session starts by reading `next.md` before any task work. A user prompt of "where did we leave off" triggers restating `next.md` verbatim.

**Reasoning:** Two failure modes the template needed to fix. (1) End-of-session drift: without an explicit hand-off, the next session starts cold and reconstructs state from scratch — slow and error-prone. (2) Multi-day discontinuity: log entries bury the forward pointer in a mass of past entries, and without a known, always-current location for "what's next," the user has to hunt for it. A dedicated file at a fixed path (`/wiki/next.md`) solves both: always findable, always overwritten, read at session start, printed at session end so the user sees it live in the chat.

**Implementation:**
- `init` skill scaffolds `/wiki/next.md` on first run with an empty hand-off.
- `init` re-run back-fills `next.md` if a project predates the contract; does not overwrite an existing one.
- `claude-template.md` teaches the resume protocol and hand-off protocol explicitly, plus the "print as closing message" rule.
- Final checklist in the template includes "`/wiki/next.md` overwritten with current hand-off, then printed as closing message."

**Rejected alternatives:**
- *"Next" section on each dated session page in `/wiki/pages/`* — no new file, but the forward pointer moves every session; you have to find the latest session page to find the pointer. Brittle when sessions nest or when multiple were opened on the same day.
- *Forward pointer inside `/wiki/log.md`* — conflates append-only history with live mutable state. Violates the log's immutability invariant.
- *A `/session-end` slash command that mechanically writes the file* — cleaner than relying on CLAUDE.md instructions, but adds a second enforcement surface (skill + template). Revisit if the instruction-based approach turns out to be unreliable in practice.

---

## D-018 — Framework/project separation: memory protocol lives in the wiki, not in CLAUDE.md

**Decision:** All rules for operating the wiki — wiki discipline, session continuity, the session hand-off contract, the memory-protocol checklist — live in a single framework-owned file, `/wiki/memory-protocol.md`, bundled with the `init` skill as `skills/init/references/memory-protocol.md`. `CLAUDE.md` keeps only project-level configuration (identity, folder structure, persona, output style, autonomy, final checklist) and a one-paragraph pointer to the protocol file. On every `init` run, `/wiki/memory-protocol.md` is created (first run) or additively merged (re-run) from the bundled reference.

**Reasoning:** Before this split, CLAUDE.md carried two concerns at once: per-project configuration (which varies by project) and the framework's memory protocol (which is identical across every project). Bundling them meant framework updates had to propagate through CLAUDE.md's additive-merge path — slow to roll out, hard to read, and architecturally muddled. Splitting gives each concern a single source of truth: project settings in CLAUDE.md, framework rules in `/wiki/memory-protocol.md`. The rules for operating the wiki now live *in* the wiki, which is conceptually consistent. Future framework edits happen in one plugin template file and propagate via `init` re-runs without touching project-specific content.

**Implementation:**
- New bundled reference: `skills/init/references/memory-protocol.md` — the canonical protocol text.
- `skills/init/references/claude-template.md` slimmed: Wiki Discipline + Session Continuity sections removed; replaced with a short `## Long-term memory protocol` pointer; Folder Structure rules bullet and Final Checklist updated to reference `/wiki/memory-protocol.md`.
- `skills/init/SKILL.md` updated: Step 3 reads three references (template, persona, memory protocol); Step 6 scaffolds `/wiki/memory-protocol.md` on every invocation (create-if-missing, additive-merge if present); Step 7 confirmation surfaces the `memory-protocol.md:` state.
- Legacy projects (CLAUDE.md still carrying the pre-extraction sections) are handled by a one-time manual cleanup before re-running `init`, not by an automatic migration path in the skill. Rationale: only one such project existed at the time of the change (Mira), so the cost of hand-cleaning it was lower than the cost of writing and maintaining migration logic.

**Rejected alternatives:**
- *Keep everything inlined in CLAUDE.md* — works, but every framework update forces a CLAUDE.md merge in every project, and the architectural split between project config and framework protocol never gets expressed.
- *Put the protocol at a plugin-owned path outside the project (e.g., read it live from the installed plugin)* — tighter canonicality, but projects would lose the ability to read the protocol without the plugin installed, and the wiki stops being self-contained. Keeping a copy inside each project's `/wiki/` preserves offline readability and the "the wiki is the project's memory" invariant.
- *Auto-migrate legacy CLAUDE.md files via a one-time skill step* — drafted in the original spec (Edit C). Dropped because only one legacy project existed; the migration logic had more surface area than the manual cleanup.

---

## D-019 — Code workspace extension: bootstrap brief CB-001

**Decision:** Implement the code-workspace extension to `yuval-core` per the design in `../output/code-workspace-design.md`, scoped by the brief at `../output/briefs/CB-001-bootstrap-code-workspace.md`. New artifacts: `init-code` skill, `write-brief` skill, `promote-to-code` command, `report-back` command. Modifications to existing `init`: scaffold `/output/briefs/`, add a "Code workspace" pointer to the slim CLAUDE.md template, extend `deprecated-sections.md` to handle the legacy memory-protocol migration through the existing sanctioned-removal channel rather than bespoke logic. Plugin version bumped accordingly.

**Reasoning:** The marketplace already supports ideation projects (`init` + wiki). Code work needs its own surface — its own git repo, its own wiki, explicit handshake protocols back to ideation. The design separates the two cleanly: ideation owns project identity at the root; code lives as a `/code/` subfolder with its own self-contained wiki and `DECISIONS.md`. Briefs (`CB-XXX`) become the only promotion-eligible artifact between modes — specs and reports remain reference material that briefs link to. Handshake commands (`promote-to-code`, `report-back`) are the only sanctioned cross-mode edges; everything else stays one-sided.

This ADR captures the decision-to-implement at the brief level. The structural sub-decisions baked into the design (nested topology, two-wiki separation, briefs-only promotion, handshake commands as the only edges, brief tags vs release tags) are recorded as their own entries (D-020 through D-024) so each is independently citable.

**Implementation:** See brief at `../output/briefs/CB-001-bootstrap-code-workspace.md` for the full scope and acceptance criteria. Bootstrap caveat: `promote-to-code` is itself part of this brief's scope, so the promotion of CB-001 was hand-authored — this ADR was added manually, the brief's status field was flipped to `promoted` in place, and the `promote` log entries were appended to both `/wiki/log.md` and `/code/wiki/log.md` (the latter back-filled after `init-code` ran).

**Rejected alternatives:**
- *Auto-create `/code/` in `init`* — pollutes ideation-only projects with unused git scaffolding. Confirmed by D-003.
- *Single wiki spanning ideation + code* — already rejected by D-001. Code-workspace design re-affirms.
- *Promote any `/output/` artifact, not just briefs* — too loose; specs and reports are reference material, not scoped commitments. Briefs are the contract.

---

## D-020 — Code workspace nested under ideation root, not sibling

**Decision:** When a project promotes to code, `/code/` is a **subfolder** of the ideation project root (`project-root/code/`), not a sibling (`project-root-code/`). Ideation owns project identity at the root. Relative paths from `/code/` to `../output/` and `../wiki/` are stable across machines and clones.

**Reasoning:** A solo operator already pays one folder coordination tax per project (D-002). Splitting into sibling top-level folders would double it. Nesting also lets `/code/` reference ideation artifacts by stable relative paths (`../output/briefs/CB-XXX.md`), which matters for ADRs that cite their originating brief. Cowork can mount either the project root (full ideation + code view) or `/code/` alone (code-only view); the inner CLAUDE.md is self-sufficient for the latter case.

**Rejected alternatives:**
- *Sibling top-level folders linked by reference* — cleanest conceptual separation but doubles setup cost and breaks stable relative paths between code and ideation artifacts.
- *Code as a git submodule of an ideation parent repo* — submodule complexity for no real benefit; D-004 already established the parent is unversioned.

---

## D-021 — Two wikis with shared protocol, each self-contained

**Decision:** Ideation and code each have their own wiki: `/wiki/` at the project root, `/code/wiki/` inside the code repo. Each carries its own full copy of `memory-protocol.md`, `index.md`, `log.md`, `next.md`, and `pages/`. No symlinks, no submodules, no shared mutable state. Both `memory-protocol.md` files are populated from the same plugin reference (`skills/init/references/memory-protocol.md`) so they stay synchronized via re-runs.

**Reasoning:** Each wiki must be functional standalone. Cowork mount of just `/code/` should be a complete code-mode workspace; mount of just the ideation root should be a complete ideation workspace. Symlinking the protocol file would break that — a missing target leaves the wiki without rules. Duplicating from a shared reference accepts a small drift risk (re-run frequency must be high enough that protocol updates propagate before they matter) in exchange for full self-containment. The wiki is the project's memory — having the rules for the wiki live *in* the wiki keeps the invariant clean.

Cross-wiki references go by relative path when needed (`[D-007](../code/DECISIONS.md#d-007)` or `[Source: foo](../../wiki/pages/foo.md)`). Link rot across the boundary is a known risk, accepted for now.

**Rejected alternatives:**
- *Single wiki spanning both modes* — rejected by D-001 originally; re-confirmed here. Audience and entity mismatch.
- *Shared `memory-protocol.md` via symlink* — fragile across OSes and breaks the standalone-wiki invariant.
- *Code wiki without its own `memory-protocol.md` (read from ideation copy)* — couples the two wikis and breaks code-only mounts.

---

## D-022 — `init-code` is a separate skill from `init`

**Decision:** Code workspace creation lives in its own skill, `init-code`, not as a flag or branch within `init`. `init` scaffolds ideation only. `init-code` scaffolds `/code/` and the code wiki, runs `git init` (or detects an existing `.git/` and skips), and populates the inner CLAUDE.md, README.md, DECISIONS.md, .gitignore, and code wiki templates.

**Reasoning:** D-003 already established `/code/` as opt-in. Putting that opt-in behind a separate skill name (rather than `init --with-code` or similar) makes the choice explicit at invocation time and keeps each skill's surface narrow and testable. Re-runs of either skill are independent: re-running `init` doesn't accidentally touch code state; re-running `init-code` doesn't churn ideation files. The design also lets `init-code` carry its own reference templates without bloating the `init` skill's bundle for projects that never need code.

**Rejected alternatives:**
- *Single `init` skill with a `--with-code` flag or interactive prompt* — couples concerns, makes re-runs ambiguous (does the flag re-apply?), and grows the skill surface.
- *Auto-detect `/code/` and run code-mode scaffolding from `init`* — surprises the user; opt-in should be explicit.

---

## D-023 — Briefs are the only promotion-eligible artifact

**Decision:** Only `CB-XXX` briefs in `/output/briefs/` are promotion-eligible via `promote-to-code`. Other `/output/` artifacts — specs, design docs, reports, decks — are reference material that briefs link to, never promoted directly. Promotion is the moment a scoped commitment crosses into code. Reference material describes the world; only briefs commit to changing it.

**Reasoning:** Anything could in principle become an ADR, but most things shouldn't. A spec is a description; an ADR is a decision. Forcing all promotion through briefs gives every code-side decision a consistent contract: a frozen, dated, scoped statement of what's being committed to. It also creates a clean lifecycle (`open` → `promoted` → `closed`) and a clean traceability path from ADR back to the originating commitment.

If a non-brief artifact contains durable insight that should reach `/code/`, the path is: author a brief that references it, then promote the brief.

**Rejected alternatives:**
- *Any `/output/` artifact can be promoted* — works mechanically but produces inconsistent ADRs (some are decisions, some are descriptions of state). Loses the contract semantics of a brief.
- *No formal promotion artifact; just file ADRs directly in `/code/DECISIONS.md`* — works but loses the bidirectional traceability between ideation scope and code commitment, and loses the freezing semantics that make a brief immutable post-promotion.

---

## D-024 — Handshake commands are the only sanctioned cross-mode edges; brief tags distinct from release tags

**Decision:** Two-part decision binding cross-mode protocol and tagging discipline.

**Cross-mode edges:** The only sanctioned mutations across the ideation/code boundary are the two handshake commands: `promote-to-code` (ideation → code) and `report-back` (code → ideation). Routine commits, file edits, and wiki updates stay within their own mode. Cross-mode events that bypass the handshakes (e.g., manually editing `/code/DECISIONS.md` to reference a `/output/` artifact without running `promote-to-code`) lose the bidirectional logging that makes the trail debuggable later.

**Tagging:** Two independent tag namespaces coexist on the code repo:
- `brief/CB-XXX` — annotated tag at the commit where a brief's acceptance criteria are met. One per closed brief. Anchors "this is the state at scope-completion of CB-XXX."
- `vX.Y.Z` — release tag, semver, independent cadence. A release may bundle multiple closed briefs; a single brief may warrant its own release.

Both can coexist on the same commit. Neither replaces the other. Release annotations should list the brief tags they include.

**Reasoning:** Without the handshake-only rule, the two wikis silently diverge — every cross-mode finding that bypasses `report-back` is invisible on the ideation side. The discipline is fragile (easy to forget) but the cost of forgetting is recoverable; the cost of allowing arbitrary cross-mode mutations is structural confusion. The two-tag-namespace decision separates "scope-completion milestone" (per-brief, frequent) from "release milestone" (per-version, less frequent). One could be derived from the other in some cases, but they answer different questions: brief tags answer "where did CB-007 land?"; release tags answer "what's in v1.4.0?".

**Rejected alternatives:**
- *Allow direct cross-mode edits without handshake commands* — loses bidirectional logging.
- *Single tag namespace (only `vX.Y.Z`, no `brief/` tags)* — releases bundle multiple briefs, so a release tag can't anchor a single brief's scope-completion. Forces brief closure into release cadence.
- *`brief/` tags only, no release tags* — works for code-only history but breaks alignment with downstream consumers expecting semver releases.

---

## D-025 — Proactive wiki protocol

**Decision:** Implement the scope captured in [`../output/briefs/CB-002-proactive-wiki-protocol.md`](../output/briefs/CB-002-proactive-wiki-protocol.md) — see brief for full to-do list, out-of-scope items, and acceptance criteria.

**Reasoning:** Update the canonical memory protocol with **proactive triggers** and the **live-session-page** convention, so wiki bookkeeping (`log.md` / `next.md` / session pages / `index.md`) happens at the right moments without requiring user prompts. Add a `Briefs` category to the `index.md` template so every brief's lifecycle state is visible at a glance.

**Brief reference:** [`../output/briefs/CB-002-proactive-wiki-protocol.md`](../output/briefs/CB-002-proactive-wiki-protocol.md) — promoted 2026-04-27, status `promoted` post-promotion. The brief is immutable post-promotion and serves as the as-of-promotion snapshot of scope.

**Rejected alternatives:** _(see brief's `## Out of scope` for what was explicitly excluded.)_
