# ai

Get the most out of coding with AI.

## Structure

- **`AGENTS.md`** — living document for non-obvious gotchas and patterns discovered while working in this codebase
- **`skills/`** — reusable agent skills following the [skills.sh](https://skills.sh) ecosystem
  - `commit/` — conventional commits (angular style)
  - `cv/` — portfolio of shipped agent skills

## Cleaning up AGENTS.md

Use this prompt to strip your `CLAUDE.md` / `AGENTS.md` down to only what matters:

```
remove everything from CLAUDE.md/AGENTS.md that can be inferred from the codebase, including high-level architecture descriptions, file trees, cli usage, build commands, and examples of standard behavior. keep only non-obvious, failure-prone decisions and hidden constraints that are not explicit in the code but would cause mistakes if misunderstood. the final file should read like a sharp-edges and gotchas document, not a project overview
```
