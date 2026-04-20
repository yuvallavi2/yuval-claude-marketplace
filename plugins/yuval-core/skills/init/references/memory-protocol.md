# Memory Protocol

_This file defines how the wiki works: what lives where, when to write to it, and how sessions hand off to each other. It is maintained by the `yuval-core:init` skill and refreshed whenever the plugin updates. Read this at the start of every session, before any wiki operation._

---

## Wiki Discipline

This project uses a persistent wiki (the Karpathy LLM Wiki pattern) as long-term memory. The wiki is the compiled, cross-referenced knowledge layer between raw sources and answers.

### Ownership

- **You own the wiki.** Write it, maintain it, keep it current.
- **The user owns `/input/`, `/in-progress/`, and `/output/`.** Never expect the user to hand-edit wiki pages.
- The user reads the wiki. You write it.

### Structure

- `/wiki/index.md` — content catalog, organized by category (Sources, Entities, Concepts, Decisions, Open Questions). Read this first when answering any question.
- `/wiki/log.md` — append-only chronological record of ingests, queries, lint passes, and sessions. Every entry starts with `## [YYYY-MM-DD] operation | title` so it's grep-able.
- `/wiki/next.md` — single-file session hand-off. Overwritten at the end of each working session, read at the start of the next. Holds the current next action, any open threads, and the one-line context needed to pick up cold.
- `/wiki/pages/` — individual wiki pages. One entity, concept, or decision per page. Cross-linked with markdown links.
- `/wiki/memory-protocol.md` — this file. The protocol itself.

### Operations (ambient vocabulary)

These operations exist as a vocabulary the user can invoke:

- **ingest** — User drops a file in `/input/` and says "ingest" (or similar). Read the source, discuss key takeaways, write a summary page to `/wiki/pages/`, update `/wiki/index.md`, update any affected existing pages, append an entry to `/wiki/log.md`. A single source may touch many pages.
- **query** — User asks a question. Read `/wiki/index.md` first, drill into relevant pages, synthesize an answer with citations to wiki pages. Good query answers should be filed back as new wiki pages when they contain durable insight.
- **lint** — Health-check the wiki. Look for contradictions, stale claims, orphan pages, important concepts lacking their own page, missing cross-references, data gaps. Report findings; propose fixes.

### When `/output/` content is also wiki-worthy

Finished deliverables live in `/output/`. If a deliverable contains durable insight (a decision, a finding, a synthesis), also create or update the corresponding wiki page. `/output/` is for the artifact; `/wiki/` is for the knowledge the artifact contains.

---

## Session Continuity

Sessions are not atomic — work crosses day boundaries. Two artifacts carry state across the gap: `/wiki/log.md` (append-only history) and `/wiki/next.md` (single-file hand-off).

### Resuming a session

At the start of every session, before any task work:

1. Read `/wiki/memory-protocol.md` (this file).
2. Read `/wiki/index.md` to orient.
3. Read `/wiki/next.md` to see where the last session left off.
4. Scan `/input/` for new or updated files.
5. If the user says "where did we leave off" (or similar), restate `/wiki/next.md` verbatim as the opening response and propose the next action from it.

### Ending a session — the hand-off

Every session that produces meaningful work does three things before wrapping up:

**1. Append to `/wiki/log.md`:**

```
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

```
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

## Memory-protocol checklist

Before wrapping a session that produced meaningful work:

- [ ] Wiki updated with any durable insight
- [ ] Log entry appended to `/wiki/log.md`
- [ ] `/wiki/next.md` overwritten with current hand-off
- [ ] Hand-off printed as closing message
