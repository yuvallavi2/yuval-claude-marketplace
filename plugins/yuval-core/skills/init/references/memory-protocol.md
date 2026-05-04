# Memory Protocol

_This file defines how the wiki works: what lives where, when to write to it, and how sessions hand off to each other. It is maintained by the `yuval-core:init` skill and refreshed whenever the plugin updates. Read this at the start of every session, before any wiki operation._

---

## Wiki Discipline

This project uses a persistent wiki (the Karpathy LLM Wiki pattern, extended into a **workshop**) as long-term memory. The wiki is the compiled, cross-referenced knowledge layer between raw sources and answers — and the surface where intent and atmosphere accumulate before they get committed into canonical artifacts.

### Workshop stance (D-029)

Canonical artifacts (`SPIRIT.md`, briefs in `/output/briefs/`, ADRs in `/code/DECISIONS.md`, code) remain authoritative. The wiki is **not** a single source of truth — it is the **workshop** where signals collect before they become commitments. Skills like `write-brief`, `refresh-spirit`, and `promote-to-code` consume the wiki as input; they do not round-trip every artifact through it.

### Ownership

- **You own the wiki.** Write it, maintain it, keep it current.
- **The user owns `/input/`, `/in-progress/`, and `/output/`.** Never expect the user to hand-edit wiki pages.
- The user reads the wiki. You write it.

### Three layers

The wiki has three layers, each with a distinct role and read cadence.

#### 1. Generative (intent, atmosphere, what's coming)

Loaded into context every session. These files are **where the next phase of the project accumulates**.

- `/wiki/vision.md` — north star. Free prose. Updated only with confirmation when framing has materially shifted.
- `/wiki/goals.md` — current-phase goals. `## Active` and `## Closed / superseded` sections. Items follow the format defined inside the file.
- `/wiki/spirit-signals.md` — raw atmospheric / voice / tone / texture signals captured between `refresh-spirit` runs. Append-only at write time. `refresh-spirit` synthesizes pending signals into `/code/SPIRIT.md` later.
- `/wiki/backlog.md` — parked tasks ("we should also someday…"). `## Active` and `## Closed` sections. Becomes a brief only via explicit `/yuval-core:write-brief` invocation.
- `/wiki/ideation/` — folder of half-formed ideas. One file per spitball: `idea-<slug>.md`.

#### 2. Retrospective (what happened)

Read on demand, not eagerly. The durable record.

- `/wiki/log.md` — append-only chronological record of ingests, queries, lint passes, sessions, brief lifecycle events. Every entry starts with `## [YYYY-MM-DD] operation | title` so it's grep-able.
- `/wiki/pages/` — individual wiki pages. One entity, concept, decision, source, session, skill, or command per page. Cross-linked with markdown links.

#### 3. State (current pointers)

Loaded into context every session. The "where am I" files.

- `/wiki/index.md` — content catalog. Read first when answering any question.
- `/wiki/next.md` — single-file session hand-off. Overwritten at the end of each working session, read at the start of the next. Holds the current next action, any open threads, and the one-line context needed to pick up cold.
- `/wiki/memory-protocol.md` — this file. The protocol itself.

### Operations (ambient vocabulary)

These operations exist as a vocabulary the user can invoke:

- **ingest** — User drops a file in `/input/` and says "ingest" (or similar). Read the source, discuss key takeaways, write a summary page to `/wiki/pages/`, update `/wiki/index.md`, update any affected existing pages, append an entry to `/wiki/log.md`. A single source may touch many pages.
- **query** — User asks a question. Read `/wiki/index.md` first, drill into relevant pages, synthesize an answer with citations to wiki pages. Good query answers should be filed back as new wiki pages when they contain durable insight.
- **lint** — Health-check the wiki. Look for contradictions, stale claims, orphan pages, important concepts lacking their own page, missing cross-references, data gaps. Report findings; propose fixes.

### When `/output/` content is also wiki-worthy

Finished deliverables live in `/output/`. If a deliverable contains durable insight (a decision, a finding, a synthesis), also create or update the corresponding wiki page. `/output/` is for the artifact; `/wiki/` is for the knowledge the artifact contains.

---

## Proactive triggers

The protocol is most effective when wiki bookkeeping happens *while* work is in flight, not at session end. Watch for these signals and act on them at the moment they occur — without waiting for the user to prompt.

### Retrospective triggers

| Trigger | Action |
|---|---|
| Artifact created or moved in `/output/` (incl. `/output/briefs/`) | Append `log.md` entry · touch the live session page |
| Architectural / scoping decision made | Append to session page under Decisions · queue for `D-xxx` promotion if the project has a code workspace |
| Brief authored / promoted / closed | `log.md` entry per the brief lifecycle (one entry per state transition) |
| Mode pivot ("moving to implementation", "let's wrap", "switching tools") | Flush wiki updates *before* pivoting |
| 5+ substantive turns since last wiki touch | Self-check; if anything durable accumulated, flush |

### Generative triggers (D-029)

| Trigger | Action |
|---|---|
| User articulates vision / restates "why we're doing this" | Read `vision.md`; if the framing has materially shifted, propose an update and **confirm before overwriting**. Vision is the only generative file with a confirmation gate. |
| User sets or shifts a phase goal | Append to `goals.md` under `## Active` (with set-date and target brief if known). Mark superseded goals as `~~struck~~ — superseded YYYY-MM-DD`. |
| User expresses an atmospheric / voice / tone preference mid-session | Append to `spirit-signals.md` as a dated raw note with status `_Status: pending synthesis._`. **Do not synthesize at write time** — `refresh-spirit` synthesizes later. |
| User says "we should also someday…" / "park this" / "remind me to" | Append to `backlog.md` as a checkbox item under `## Active`. |
| User spitballs an unfinished idea | Create a new file in `ideation/` named `idea-<slug>.md`. Free-form body — no schema. |

### Live session page

A dated session page in `/wiki/pages/` (e.g., `session-YYYY-MM-DD-topic.md`) is the **running notebook** of the session, not a closing deliverable.

- **Create on the first durable event** of the session — first artifact, first decision, first milestone choice — whichever comes first. Don't wait for session end.
- **Append throughout** the session under topical subheadings (e.g., `## Decisions`, `## Artifacts`, `## Open threads`). The page grows live.
- **Finalize at session end** as part of the existing hand-off ritual — add a closing summary, link from `/wiki/index.md` if the project's index has a Sessions category.

### Surfacing convention

Wiki bookkeeping should be auditable without becoming chatter:

- **Silent** for minor wiki touches: single log line, status field updates, routine appends, generative-trigger captures.
- **One-line acknowledgment** in chat at major milestones only: artifact created in `/output/`, brief authored / promoted / closed, mode pivot completed, vision update proposed.
- **No multi-paragraph reports** of bookkeeping. The wiki itself is the record; the chat is for working.

---

## Session Continuity

Sessions are not atomic — work crosses day boundaries. Two artifacts carry state across the gap: `/wiki/log.md` (append-only history) and `/wiki/next.md` (single-file hand-off).

### Resuming a session

At the start of every session, before any task work:

1. Read `/wiki/memory-protocol.md` (this file).
2. Read `/wiki/index.md` to orient.
3. Read `/wiki/next.md` to see where the last session left off.
4. Read `/wiki/vision.md` to load the north star into context.
5. Read `/wiki/goals.md` to load current-phase goals into context.
6. Read `/wiki/spirit-signals.md` to load any pending atmosphere signals into context.
7. Scan `/input/` for new or updated files.
8. If the user says "where did we leave off" (or similar), restate `/wiki/next.md` verbatim as the opening response and propose the next action from it.

The three generative reads (vision / goals / spirit-signals) load **project character** into context every session, not just retrospective state.

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

- [ ] Wiki updated with any durable insight (retrospective layer)
- [ ] Generative-trigger captures applied (vision / goals / spirit-signals / backlog / ideation as appropriate)
- [ ] Log entry appended to `/wiki/log.md`
- [ ] `/wiki/next.md` overwritten with current hand-off
- [ ] Hand-off printed as closing message
