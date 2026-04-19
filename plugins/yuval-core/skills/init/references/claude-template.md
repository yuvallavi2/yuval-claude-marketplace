# CLAUDE.md — Project Workspace Instructions

---

## Project Identity
- **Project Name:** {{PROJECT_NAME}}
- **Created:** {{DATE}}
- **Type:** {{PROJECT_TYPE}}
- **Goal:** {{PROJECT_GOAL}}

---

## Folder Structure

Always respect this structure — no exceptions.

```
/input/          → Raw sources. Immutable. User owns this.
/in-progress/    → Working drafts and scratch. Deletable at will.
/output/         → Final deliverables only — clean, complete, ready to ship.
/wiki/           → Long-term memory. LLM owns this. User reads it.
```

### Rules
- **Start every session** by scanning `/input/` for new or updated files, reading `/wiki/index.md` to orient, and reading `/wiki/next.md` to pick up the last session's hand-off.
- **Never write directly to `/output/`** until the deliverable is complete.
- **Never delete** files from `/input/` or `/wiki/` without explicit instruction.
- `/in-progress/` is the only folder the user treats as deletable. Don't put anything there you expect to survive.

---

## Wiki Discipline

This project uses a persistent wiki (the Karpathy LLM Wiki pattern) as long-term memory. The wiki is the compiled, cross-referenced knowledge layer between raw sources and answers.

### Ownership

- **You own the wiki.** Write it, maintain it, keep it current.
- **The user owns `/input/`, `/in-progress/`, and `/output/`.** Never expect the user to hand-edit wiki pages.
- The user reads the wiki. You write it.

### Structure

- `/wiki/index.md` — content catalog, organized by category (Sources, Entities, Concepts, Decisions, Open Questions). Read this first when answering any question.
- `/wiki/log.md` — append-only chronological record of ingests, queries, and lint passes. Every entry starts with `## [YYYY-MM-DD] operation | title` so it's grep-able.
- `/wiki/next.md` — single-file session hand-off. Overwritten at the end of each working session, read at the start of the next. Holds the current next action, any open threads, and the one-line context needed to pick up cold.
- `/wiki/pages/` — individual wiki pages. One entity, concept, or decision per page. Cross-linked with markdown links.

### Operations (ambient vocabulary)

These operations exist as a vocabulary the user can invoke. The full implementation may come from a dedicated `llm-wiki` skill, but the behavior should work ambiently:

- **ingest** — User drops a file in `/input/` and says "ingest" (or similar). Read the source, discuss key takeaways, write a summary page to `/wiki/pages/`, update `/wiki/index.md`, update any affected existing pages, append an entry to `/wiki/log.md`. A single source may touch many pages.
- **query** — User asks a question. Read `/wiki/index.md` first, drill into relevant pages, synthesize an answer with citations to wiki pages. Good query answers should be filed back as new wiki pages when they contain durable insight.
- **lint** — Health-check the wiki. Look for contradictions, stale claims, orphan pages, important concepts lacking their own page, missing cross-references, data gaps. Report findings; propose fixes.

### When `/output/` content is also wiki-worthy

Finished deliverables live in `/output/`. If a deliverable contains durable insight (a decision, a finding, a synthesis), also create or update the corresponding wiki page. `/output/` is for the artifact; `/wiki/` is for the knowledge the artifact contains.

---

<!-- PERSONA:START -->
<!-- Replaced at init time with content from persona.md -->
<!-- PERSONA:END -->

---

## Project Types Reference

| Type | Focus |
|---|---|
| **Product/Strategy** | Competitive analysis, pricing, licensing, lifecycle planning, roadmaps |
| **Technical Research** | MCP architecture, agentic systems, tool design |
| **Content Creation** | LinkedIn posts, executive decks, presentations — punchy, first-person, human |
| **Customer/Market Analysis** | Segment analysis, territory planning, partner ecosystems |
| **Brainstorm/Exploration** | Open-ended thinking, early-stage ideation, pre-PRD work |

---

## Default Output Style

**Lead with visuals when they clarify.** For complex deliverables:
1. A diagram, flowchart, or structured visual capturing the core idea
2. A concise written explanation beneath it
3. Bullet-point action items or next steps at the end

Use diagrams for flows and architectures. Use tables for comparisons.
For presentations and LinkedIn content: punchy, first-person, conversational.

For conversational answers, prose. Visuals are a tool, not a requirement.

---

## Autonomy & Work Style

**Full autopilot once a task is defined:**
- Do not ask clarifying questions mid-task unless something would cause irreversible harm
- State assumptions briefly at the start, then proceed
- Complete the full task end-to-end, then present results
- If multiple approaches exist, pick the best one and note alternatives at the end

**Check in only when:**
- A file would be permanently deleted or overwritten
- Task scope is significantly larger than implied
- A blocking ambiguity exists that no assumption can resolve

---

## Session Continuity

Sessions are not atomic — work crosses day boundaries. Two artifacts carry state across the gap: `/wiki/log.md` (append-only history) and `/wiki/next.md` (single-file hand-off).

### Resuming a session

At the start of every session, before any task work:

1. Read `/wiki/index.md` to orient.
2. Read `/wiki/next.md` to see where the last session left off.
3. Scan `/input/` for new or updated files.
4. If the user says "where did we leave off" (or similar), restate `/wiki/next.md` verbatim as the opening response and propose the next action from it.

### Ending a session — the hand-off

Every session that produces meaningful work does two things before wrapping up:

**1. Append to `/wiki/log.md`:**

```markdown
## [YYYY-MM-DD] session | Brief description
- Task: [what was asked]
- Assumptions: [list]
- Files created/modified: [list]
- Output delivered: [filename or description]
- Wiki pages touched: [list]
- Open items: [anything left incomplete]
```

The `## [YYYY-MM-DD] operation |` prefix is grep-friendly — `grep "^## \[" wiki/log.md | tail -5` returns the last 5 entries.

**2. Overwrite `/wiki/next.md` with the current hand-off:**

```markdown
# Next — [PROJECT_NAME]

_Last updated: YYYY-MM-DD_

## Next action
[One concrete thing to do next. A verb, not a vibe.]

## Why
[One or two sentences of context so future-you can pick up cold without rereading the whole log.]

## Open threads
- [Anything unresolved, blocking, or waiting on the user]

## Recent context
- Last session: [one-line summary, link to session page if there is one]
```

**3. Print the contents of `/wiki/next.md` as the closing message of the session** — the "hot swap." The user sees the hand-off in the chat, can act on it immediately, and the same hand-off is persisted for the next cold start.

---

## Final Checklist Before Writing to /output/
- [ ] Deliverable fully addresses the task
- [ ] Visual/diagram included where applicable
- [ ] Tone matches the project type
- [ ] Wiki updated with any durable insight from this work
- [ ] Log entry appended to `/wiki/log.md`
- [ ] `/wiki/next.md` overwritten with current hand-off, then printed as closing message
- [ ] No draft files in `/output/`
