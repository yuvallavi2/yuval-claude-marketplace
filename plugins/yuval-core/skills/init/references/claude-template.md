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
- **Start every session** by reading `/wiki/memory-protocol.md`, then `/wiki/index.md`, then `/wiki/next.md`. Then scan `/input/` for new or updated files.
- **Never write directly to `/output/`** until the deliverable is complete.
- **Never delete** files from `/input/` or `/wiki/` without explicit instruction.
- `/in-progress/` is the only folder the user treats as deletable. Don't put anything there you expect to survive.

---

## Long-term memory protocol

All rules for operating the wiki, session continuity, and the session hand-off live in `/wiki/memory-protocol.md`. Read that file at session start, before any wiki operation. It is the single source of truth for the memory protocol.

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

## Final Checklist Before Writing to /output/
- [ ] Deliverable fully addresses the task
- [ ] Visual/diagram included where applicable
- [ ] Tone matches the project type
- [ ] Memory-protocol checklist in `/wiki/memory-protocol.md` satisfied
- [ ] No draft files in `/output/`
