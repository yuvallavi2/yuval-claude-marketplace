# Spirit Signals — {{PROJECT_NAME}}

_Raw atmospheric / voice / tone / texture signals captured between `refresh-spirit` runs. Append-only at write time. `refresh-spirit` synthesizes pending signals into `/code/SPIRIT.md` later._

**Format.** Each signal is a `## YYYY-MM-DD — <one-line title>` section. Body is whatever the user said (paraphrased or quoted), plus any context for why it matters. Footer is a status line:

- `_Status: pending synthesis._` — captured but not yet folded into SPIRIT.md
- `_Status: synthesized into SPIRIT.md on YYYY-MM-DD via /yuval-core:refresh-spirit._` — applied; kept here as the audit trail

Do **not** try to synthesize at write time. Capture the raw signal and move on. Synthesis is `refresh-spirit`'s job.

If the project's SPIRIT.md is `n/a` (pure-tooling, no user-facing surface), this file is also `n/a` — keep the file for consistency, leave the body empty.

---

_None yet._
