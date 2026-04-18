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
