---
name: refresh-persona
description: >
  Re-sync the persona block in an already-initialized project's CLAUDE.md from the bundled
  persona.md, without re-running init. Use when the user says "refresh persona", "update
  persona", "sync persona", "pull latest persona", or otherwise asks to refresh the
  CLAUDE.md persona block in an existing project. Replaces only the content between the
  <!-- PERSONA:START --> and <!-- PERSONA:END --> markers; everything else in CLAUDE.md is
  byte-for-byte preserved.
---

# Refresh Persona

When triggered, execute all steps autonomously with no clarifying questions. The user's selected folder is the project root.

This skill ships the deferred half of D-007: persona evolves independently of `init`, and a separate path exists to pull persona updates into projects that already have a `CLAUDE.md`. It performs a single, narrow mutation — replace the content between `<!-- PERSONA:START -->` and `<!-- PERSONA:END -->` with the bundled persona — and nothing else.

---

## Step 1 — Locate `CLAUDE.md`

Look for `CLAUDE.md` in the project root.

**If `CLAUDE.md` is missing**, abort with:

> No `CLAUDE.md` in this project. Run `/yuval-core:init` first.

Do not create or scaffold anything. Stop.

## Step 2 — Locate the persona markers

Read the existing `CLAUDE.md`. Search for both `<!-- PERSONA:START -->` and `<!-- PERSONA:END -->`.

**If either marker is missing**, abort with:

> Persona markers not found in `CLAUDE.md`. This project predates D-007 and needs a one-time manual cleanup before refresh can run — add the marker pair around the existing persona block, or re-run `/yuval-core:init` against a fresh project.

Do **not** auto-insert markers. Auto-insertion has unbounded collision risk with user-edited content; the user does the one-time cleanup, then refresh becomes mechanical from then on.

## Step 3 — Read the bundled persona

Read `${SKILL_DIR}/references/persona.md` using the Read tool.

**If the file is missing**, this is a broken plugin install. Stop and warn the user.

Strip the file's preamble — the top-level `# Persona — …` heading and any HTML-comment block at the top — to produce the **persona body**. The persona body is what gets inserted between the markers (the markers themselves remain).

## Step 4 — Compute current and new persona blocks

- **Current block:** the text between `<!-- PERSONA:START -->` and `<!-- PERSONA:END -->` in `CLAUDE.md`, exclusive of the markers themselves.
- **New block:** the persona body from Step 3.

Normalize trailing whitespace before comparison (a stray blank line should not force a write).

**If the two are identical**, exit with:

> Persona is already up to date.

Write nothing. Stop.

## Step 5 — Show a unified diff

Print a unified diff (`diff -u` style) of the two blocks so the user can scan it. No confirmation prompt — proceed straight to the write per workbench autopilot norm.

## Step 6 — Write the replacement

Replace the content between `<!-- PERSONA:START -->` and `<!-- PERSONA:END -->` with the new persona body. Constraints:

- Keep the markers themselves byte-for-byte unchanged.
- Keep all `CLAUDE.md` content outside the markers byte-for-byte unchanged.
- Preserve the project's original line-ending style.

The `Edit` tool is suitable here: match the marker pair plus the current persona body as `old_string`, replace with the marker pair plus the new persona body as `new_string`.

## Step 7 — Confirm

Respond with:

```
Persona refreshed: [path to CLAUDE.md]
Source: skills/refresh-persona/references/persona.md
Block replaced between <!-- PERSONA:START --> and <!-- PERSONA:END -->; rest of CLAUDE.md byte-for-byte unchanged.
```

---

## Notes for the implementer

- **Whole-block replacement is the contract.** Per the brief, users who customize the persona block forfeit those edits on refresh, by design. There is no per-section preservation logic.
- **No backup.** Git is the version control system. Writing a `.bak` would be redundant and littering.
- **No `--dry-run` flag.** The Step 5 diff already provides visibility before the Step 6 write; a separate dry-run mode would be feature creep at v0.
- **Drift with `init`'s persona.** This skill carries its own copy of `persona.md` per D-016. The two copies (init + refresh-persona) may drift; that is tolerated, not policed. If drift becomes a real problem, author a successor brief.
