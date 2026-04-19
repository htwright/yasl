# YASL

**Yet Another Skill Library — but this one's a pipeline.**

A spec-driven development workflow for [Claude Code](https://claude.com/claude-code). Chained skills that turn a vague idea into production code, with context that survives across sessions.

For the philosophy behind this pipeline, see [VISION.md](VISION.md).

> **Working title.** Name subject to change before v1.0.

---

## The problem

Most AI coding is vibes. You describe a thing, Claude writes it, you review, repeat. It works for small tasks. It falls apart when:

- The problem is fuzzy and needs to be discovered, not just executed.
- The work spans multiple sessions and context evaporates between them.
- Claude quietly re-architects during implementation when an assumption breaks.
- A team member picks up where you left off and can't tell what was decided vs. assumed.

## The pipeline

YASL is a chain of skills. Each skill's output is structurally the next skill's input. Artifacts on disk are the memory.

```
/ideate → /spec → /plan → /build → /validate → /review
```

| Skill | Input | Output |
|---|---|---|
| `/ideate` | A fuzzy problem | A bounded problem map |
| `/spec` | A problem map (or a clear ask) | A feature spec with architecture + contracts |
| `/plan` | A spec | A step-by-step, file-by-file milestone plan |
| `/build` | A plan | Code, with checkpoint commits at each phase |
| `/validate` | Any state | A report from type-check / lint / tests |
| `/review` | A feature spec + its build | A completion report against the spec |

Plus support skills: `/prd`, `/commit`, `/setup`.

### Why the chain matters

- **Handoff notes.** Each artifact has a `Handoff Notes for /<next>` section — soft context that doesn't fit the formal structure but would be lost otherwise.
- **Decision logs.** Spec and plan each carry a decision log. `/build` reads it before making judgment calls, so it doesn't re-litigate during implementation.
- **Backward flow.** When `/build` finds an assumption the spec got wrong, it stops and kicks back to `/spec` — no silent re-architecting in code.
- **Checkpoint commits.** `/build` commits `checkpoint(<feature>): step N` with greppable trailers. Kill the session, open a new one, `/build` finds the checkpoint and resumes.

## When to use it

**Good fit:**
- Meaty features that would take an engineer several hours.
- Work that crosses multiple files or component boundaries.
- Problems that are still fuzzy — you need to discover the shape, not just execute.
- Teams where multiple people (or multiple sessions) need to pick up the same thread.

**Overkill for:**
- One-file bug fixes.
- Throwaway scripts.
- Work where the spec is obvious and well-scoped — go straight to implementation.

## Install

_Coming with v1.0._ For now:

1. Clone this repo.
2. Copy `.claude/skills/*` into your project's `.claude/skills/`.
3. Run `/setup` to generate `.claude/yasl.config.md` with your project's values.

A Claude Code plugin marketplace listing is planned.

## Configuration

After `/setup`, your project has a `.claude/yasl.config.md` file with the four things YASL needs to know about your repo:

- **Commands** — how to type-check, lint, test.
- **Commit scopes** — for `/commit`'s suggestions.
- **Example patterns** — files `/plan` can point at as templates.
- **Paths** — where artifacts live (`docs/` by default).

Skill bodies stay pristine. When you upgrade YASL, your config survives.

## Project status

**Pre-v1.** Skill bodies are being swept clean of project-specific hardcodes. The first dogfood project is [RowHouse-Run](https://github.com/htwright/RowHouse-Run). Worked examples, a setup skill, and a plugin marketplace listing are all on the runway before v1.0.

## License

MIT.
