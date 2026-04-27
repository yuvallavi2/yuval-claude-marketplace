# {{PROJECT_NAME}}

Code repo for the **{{PROJECT_NAME}}** project. The product/strategy thinking that shaped this codebase lives one level up in `../output/` and `../wiki/`; this folder is the implementation.

## Layout

```
CLAUDE.md       → code-mode rules for AI assistants
DECISIONS.md    → ADR log (architectural decisions with rationale)
wiki/           → code-side knowledge wiki (architecture, modules, patterns, bugs)
```

## Where decisions come from

Most ADRs in `DECISIONS.md` were promoted from briefs in `../output/briefs/`. Each ADR cites its originating brief by relative path. Briefs are immutable post-promotion — they record what was scoped at the moment of commitment.

## Workflow

- Author a brief in `../output/briefs/` with `/yuval-core:write-brief`.
- Promote it with `/yuval-core:promote-to-code CB-XXX` — adds an ADR, freezes the brief.
- Implement; commit continuously; close the brief with an annotated `brief/CB-XXX` tag at the scope-completion commit.
- If implementation surfaces something that changes ideation, push it back with `/yuval-core:report-back`.
