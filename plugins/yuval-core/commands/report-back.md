---
description: Push a finding from code-mode back to ideation — logs both wikis, optionally touches a /wiki/pages/ page.
argument-hint: "<finding>" [CB-XXX] [--detail "<detail>"]
---

# /yuval-core:report-back

Reverse handshake: code → ideation. Use when implementation surfaces a fact, decision, or constraint that ideation needs to know. Examples: "the migration approach the brief assumed doesn't work for the current dataset shape", "the API rate limit is 200/sec, not 500", "we discovered the auth flow has a fourth state we hadn't documented."

**Without `report-back`, the two wikis silently diverge** — code learns things ideation never hears about. This command is the only sanctioned channel.

---

## Argument modes

`$ARGUMENTS` can be passed in two ways:

- **One-line argument:** `/yuval-core:report-back "<finding>"` — concise, fast. Optionally followed by a `CB-NNN` brief ID and `--detail "<detail>"`.
- **Empty argument:** `/yuval-core:report-back` — interactive. Prompt for finding, then optional detail, then optional brief ID.

If the user passes only a quoted finding, that's enough — proceed without prompting for the rest.

---

## Refusal cases

1. **No `/code/` workspace.** If `/code/` doesn't exist, refuse: *"`report-back` is a code → ideation handshake. There's no code workspace yet — this command applies once `/code/` exists. Did you mean to update `/wiki/` directly?"*
2. **No `/wiki/` (ideation root).** If the project root has no `/wiki/`, the project isn't initialized as ideation. Refuse: *"This project doesn't have an ideation `/wiki/`. Run `/yuval-core:init` at the project root first."*

---

## Steps

### 1. Collect inputs

Required:
- `FINDING` — one-line statement of what was learned. If not in `$ARGUMENTS`, prompt: *"What did the code session surface? — one line."*

Optional:
- `DETAIL` — multi-sentence elaboration. If not in `$ARGUMENTS` or via `--detail`, prompt once: *"Anything to add (rationale, evidence, or links)? Press enter to skip."*
- `CB_ID` — the brief in flight, if any. If not in `$ARGUMENTS` and there's an obviously-active brief in `/output/briefs/` (status `promoted`), prompt with that as the default: *"Reporting against `CB-NNN`? (enter to confirm, or type a different ID, or `none`)"*. If only one `promoted` brief exists, suggest it; if multiple, list them; if none, default to `none`.

### 2. Decide whether a `/wiki/pages/` page is touched

If the finding is durable (a fact about the world that future sessions will need), it should land on a wiki page. Determine:

- **Existing page that fits?** Search `/wiki/pages/` for a page matching the topic. If a strong match exists, append a dated note to it. Format:
  ```markdown

  ### Update [{{DATE}}] — {{FINDING}}
  {{DETAIL}}
  _Reported back from code session{{ , brief CB-NNN if any }}._
  ```
- **No matching page; finding is durable?** Prompt: *"This looks worth a wiki page. Want me to create `/wiki/pages/<slug>.md`?"* If yes, create it with the finding as the page body. If no, skip.
- **Finding is ephemeral or just a status update?** Skip the page step. Log entries alone are enough.

Track which page (if any) was touched — pass to the log entries.

### 3. Append to `/wiki/log.md` (ideation)

```markdown
## [{{DATE}}] report-back | {{FINDING}}
- From: code session{{ , brief CB-NNN if any }}
- Detail: {{DETAIL or "—"}}
- Wiki pages touched: {{PAGE_LIST or "none"}}
```

### 4. Append mirror entry to `/code/wiki/log.md`

```markdown
## [{{DATE}}] report-back | {{FINDING}}
- To: ideation /wiki/
- Detail: {{DETAIL or "—"}}
- Brief in flight: {{CB_ID or "none"}}
- Ideation pages touched: {{PAGE_LIST or "none"}}
```

The mirror entry is what makes the trail debuggable from either side. Without it, code-side history loses any signal that ideation was ever updated.

### 5. Confirm

```
Reported back: {{FINDING}}
Logged in:
  - /wiki/log.md (ideation)
  - /code/wiki/log.md (code)
Wiki pages touched: {{PAGE_LIST or "none"}}
Brief in flight: {{CB_ID or "none"}}
```

---

## When to invoke

Trigger `report-back` when code-side work surfaces:

- A constraint that invalidates an assumption in the active brief.
- A new entity, system, or concept that should appear in `/wiki/index.md`.
- A correction to something already in `/wiki/pages/`.
- A finding that should change ideation's next action even if no brief is in flight.

Do **not** invoke for:

- Routine implementation progress — that's `git log` and the periodic `## [DATE] session | ...` entry in `/code/wiki/log.md`.
- Clarifying questions back to the user — those are conversational, not durable.

If a `report-back` is **scope-breaking** for the active brief (the finding makes the brief's scope wrong), close the current brief with a status note explaining why, return to ideation, and author a successor brief that references the closed one as predecessor. See `/output/code-workspace-design.md` §5c for the loop pattern.
