---
name: refresh-spirit
description: >
  Re-walk the 8 spirit prompts and overwrite a project's `/code/SPIRIT.md` from the
  user's updated answers. Use when the user says "refresh spirit", "update spirit",
  "re-do SPIRIT.md", "the project's atmosphere has changed", or otherwise asks to
  evolve the project-level spirit contract. Mirrors the refresh-persona pattern
  (D-026, D-027): pre-fills each prompt with current SPIRIT.md content for editing,
  shows a unified diff, then overwrites. Aborts cleanly if `/code/SPIRIT.md` does
  not yet exist.
---

# Refresh Spirit

When triggered, execute autonomously. The user's selected folder is the project root.

This skill is the dedicated path for evolving project atmosphere after the first brief has authored `/code/SPIRIT.md`. It mirrors the contract of `refresh-persona`: a single, narrow mutation — re-walk the 8 prompts pre-filled with current content, show a diff, overwrite — and nothing else.

Per D-028, project spirit is set once and refreshed via this skill. Briefs reference SPIRIT.md; they do not redefine it. If a brief's per-brief override would balloon past one paragraph, that's a signal to run this skill instead of inlining the override.

<!--
Skill-wiki contract (D-030):
- Declared READS:  /code/SPIRIT.md, /wiki/spirit-signals.md
- Declared WRITES: /code/SPIRIT.md (canonical, whole-file replacement, user-confirmed via diff), /wiki/spirit-signals.md (status-line append only — never content edits, never deletions)
- Sparse-wiki fallback: missing /wiki/spirit-signals.md or no pending entries → existing flow runs unchanged. Wiki read is additive, not gating.
-->

---

## Step 1 — Locate `/code/SPIRIT.md`

Look for `/code/SPIRIT.md` at the workspace root.

**If it is missing**, abort with:

> No `/code/SPIRIT.md` exists yet. Run `/yuval-core:write-brief` to author it as part of your next brief, or just answer the spirit prompts directly.

Do not create or scaffold anything. Do not walk the prompts in author-mode. Stop. Authoring SPIRIT.md from nothing is `write-brief`'s job (or a deliberate manual edit by the user); this skill only refreshes existing content.

## Step 2 — Read the bundled prompts

Read `${SKILL_DIR}/references/spirit-prompts.md` using the Read tool.

**If the file is missing**, this is a broken plugin install. Stop and warn the user.

The 8 prompts and their example answers are the same set used by `write-brief` (per D-016, this skill carries its own copy; cross-skill drift is tolerated, not policed).

## Step 3 — Read the current SPIRIT.md

Read `/code/SPIRIT.md`. Parse it loosely — the file is human prose, not structured data, so don't fight the format. Identify the user's existing answer for each of the 8 prompts (where present). Track which prompts have content vs which are absent vs which are explicitly `n/a`.

If the file is the single-line `n/a — pure infrastructure project. ...` form, treat all 8 prompts as currently `n/a` and walk each one fresh.

## Step 3.5 — Read pending spirit-signals (D-030)

Read `/wiki/spirit-signals.md` if it exists.

- **File missing:** silently skip — no signals to surface, no behavior change. Continue to Step 4.
- **File present, no `_Status: pending synthesis._` entries:** silently skip. Continue.
- **File present with one or more pending entries:** parse the `## YYYY-MM-DD — <title>` sections whose footer is `_Status: pending synthesis._` (skip already-synthesized signals — they're history). Surface them to the user before the prompt walk:

  > *"Since the last refresh, <N> spirit signals have accumulated:*
  > *- YYYY-MM-DD — <title>: <one-line gist>*
  > *- YYYY-MM-DD — <title>: <one-line gist>*
  >
  > *I'll fold these into the pre-fills below."*

  Stash the parsed pending signals (date + title + body). For each of the 8 prompts in Step 4, when computing the pre-fill, **read the current SPIRIT.md answer AND the relevant pending signal(s)** to compose the proposed pre-fill text. Use Claude judgment to decide which signals are relevant to which prompt — there's no fixed mapping.

The user can still keep / edit / replace / set to `n/a` per the existing Step 4 contract — pre-fills are suggestions, not impositions.

Track the list of pending-signal section headings (e.g., `## 2026-05-04 — voice-first-warmth`) for Step 7.5.

## Step 4 — Walk the 8 prompts, pre-filled

Ask **one prompt at a time** (per workbench user-collaboration norms). For each prompt:

1. Show the prompt text from `references/spirit-prompts.md`.
2. Show the user's current answer (from Step 3) as the pre-fill.
3. Ask: *"Keep, edit, or replace? (or `n/a`)"*

Allow the user to:
- Keep the current answer unchanged.
- Edit it (the user types the new version; you accept it as-is).
- Replace it entirely with a new answer.
- Set it to `n/a` (a previously-answered prompt is being retired).

The 8 prompts in order:

1. Reference Scenario
2. Interaction Shape
3. Voice / Tone samples
4. Anti-Shape
5. Silent State
6. Tactile Contract
7. Default Posture
8. The One Moment

`n/a` is valid at any prompt, including for prompts that previously had content — the project's atmosphere may genuinely have evolved away from a previous answer.

## Step 5 — Compose the new SPIRIT.md

Build the new file content:

- **If all 8 final answers are `n/a`**, the new content is the single line:
  ```
  n/a — pure infrastructure project. No user surface, no atmosphere to transmit.
  ```
- **Otherwise**, build a markdown file with one `## <prompt name>` heading per non-`n/a` answer, and a top frontmatter block:
  ```markdown
  # SPIRIT — <project name>

  _Authored: <preserve original date if present, otherwise leave the existing line intact> | Last refreshed: YYYY-MM-DD via `/yuval-core:refresh-spirit`._

  <body: one section per non-n/a prompt>
  ```

Preserve the project name from the existing file's H1 if present.

## Step 6 — Show a unified diff

Print a `diff -u`-style diff of old vs new SPIRIT.md so the user can scan it. No confirmation prompt — proceed straight to the write per workbench autopilot norm.

If the new content is byte-identical to the old (the user kept all answers unchanged), write nothing to SPIRIT.md. **However:** if Step 3.5 captured pending signals, still run Step 7.5 — the user explicitly chose to keep all answers, which is itself a confirmation that the existing synthesis covers the new signals. Then exit with:

> SPIRIT.md is already up to date. <N> spirit signals marked synthesized.

(Or simply *"SPIRIT.md is already up to date."* if no pending signals were captured.) Stop.

## Step 7 — Overwrite SPIRIT.md

Write the new content to `/code/SPIRIT.md`. Whole-file replacement is the contract — there is no per-section preservation logic. Git is the version control system; no `.bak` file.

## Step 7.5 — Mark spirit-signals synthesized (D-030)

If Step 3.5 captured pending signals, update `/wiki/spirit-signals.md` now. For each previously-pending section (tracked in Step 3.5), replace its `_Status: pending synthesis._` footer line with:

```
_Status: synthesized into SPIRIT.md on YYYY-MM-DD via /yuval-core:refresh-spirit._
```

(Substitute today's date.)

**Append-only contract.** Never delete a signal section. Never edit body text. Only the status footer line is mutated. The full history of raw signals + when each was synthesized is the audit trail; deleting a signal would erase that.

If Step 3.5 found no pending signals (or `/wiki/spirit-signals.md` doesn't exist), skip this step entirely.

(The byte-identical SPIRIT.md case is handled inline in Step 6 — Step 7.5 is invoked there and the skill exits early without running Step 7.)

## Step 8 — Confirm

Respond with:

```
SPIRIT.md refreshed: /code/SPIRIT.md
Source: 8 prompts at skills/refresh-spirit/references/spirit-prompts.md
<count> prompts updated · <count> kept · <count> set to n/a
Spirit signals folded in: <count> (status flipped to synthesized in /wiki/spirit-signals.md)
```

(Omit the last line if no pending signals were captured in Step 3.5.)

---

## Notes for the implementer

- **Mirrors refresh-persona (D-026, D-027).** Same skill shape, same abort-on-missing stance, same diff-then-overwrite flow. The decisions baked into refresh-persona apply here too: skill not flat command file (per D-016); abort with guidance on missing source rather than auto-creating.
- **No `--dry-run` flag.** The Step 6 diff already provides visibility before the Step 7 write.
- **No schema validation.** SPIRIT.md is human prose; "exists with content" is the only check. Don't try to enforce structure beyond the loose `## <prompt name>` heading convention.
- **Drift with write-brief's spirit-prompts.md.** Per D-016, each skill carries its own `references/`. The two copies (write-brief + refresh-spirit) may drift; that is tolerated, not policed. If drift becomes a real problem, author a successor brief.
- **Whole-file replacement is the contract.** Per the brief, users who hand-edit SPIRIT.md outside the skill forfeit those edits on refresh, by design. There is no merge logic.
- **Authoring vs refreshing are different surfaces.** `write-brief` authors SPIRIT.md from nothing on the first brief; `refresh-spirit` only refreshes existing content. Keep the surfaces narrow.
