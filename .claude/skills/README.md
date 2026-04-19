# YASL Skills

The YASL pipeline, as a set of Claude Code skills.

```
/ideate → /spec → /plan → /build → /validate → /review
```

## Commands

| Command | Purpose | Output |
|---|---|---|
| `/start [task]` | Recommend where to enter the pipeline | Recommendation (no artifact) |
| `/ideate [topic]` | Explore and bound a problem space | `docs/ideation/<topic>.md` |
| `/prd [amend]` | Create or update product requirements | `docs/PRD.md` |
| `/spec <feature>` | Define requirements + architecture | `docs/specs/<feature>.md` |
| `/breakdown <feature>` | Assess scope, propose milestones | `docs/plans/<feature>/overview.md` |
| `/prime [feature]` | Load context before planning | Context summary |
| `/plan <feature> [N]` | Create executable plan for a milestone | `docs/plans/<feature>/milestone-N.md` |
| `/build <feature> [N]` | Execute plan with checkpoints | Code + updated plan |
| `/commit [msg] [opts]` | Stage, commit, push (conventional) | Git commit |
| `/validate [scope]` | Run automated tests | Validation report |
| `/review <feature>` | Validate against spec | Completion report |

Plus `/setup` — bootstraps `.claude/yasl.config.md` for your project. _(Not yet shipped — run manually for now using `templates/yasl.config.md`.)_

## How the pipeline flows

- `/ideate` produces a **problem map** → `/spec` reads it as a fixed constraint
- `/spec` produces a **spec + solution map + decision log** → `/plan` reads both for decomposition
- `/plan` produces a **milestone plan** → `/build` executes it step by step
- `/build` commits `checkpoint(<feature>): step N ...` with greppable trailers → resumable across sessions
- `/validate` and `/review` run from the same config — different questions, same commands

Each artifact carries **Handoff Notes** for the next phase. Context survives between sessions because it lives in files, not memory.

## Transition modes

`/ideate` and `/spec` operate in one of three modes (MANUAL / ADVISORY / ASSERTIVE) that govern how aggressively they propose convergence. Start in MANUAL; raise to ADVISORY or ASSERTIVE once you trust the system. Switch mid-conversation with "switch to advisory" or similar.

## Backward flow

When `/build` finds a spec assumption that doesn't hold, it **stops** — sets plan status to `BLOCKED` and kicks back to `/spec`. It does not silently re-architect. Same pattern applies at every boundary: `/spec` returns to `/ideate` if it hits a problem-space question.

## Configuration

Each skill reads `.claude/yasl.config.md` at startup to find:

- Commands (`type_check`, `lint`, `test`, optional `test_secondary` and `test_production`, optional `dev`)
- Commit scopes (for `/commit`)
- Example-pattern files (for `/plan` and `/prime`)
- Docs root path

Skill bodies are otherwise project-agnostic. Upgrading YASL means pulling new skill versions — your config survives.

## File structure the skills maintain

```
docs/
├── ideation/<topic>.md         # problem maps from /ideate
├── solutions/<feature>.md      # solution maps from /spec
├── specs/<feature>.md          # feature specs from /spec
└── plans/<feature>/
    ├── overview.md             # milestone breakdown (if decomposed)
    ├── milestone-1.md          # executable plan
    └── milestone-2.md
```

Plus `docs/PRD.md` and `docs/BACKLOG.md` if your project uses them.

## When to use, when to skip

**Use for:** meaty features, fuzzy problems, cross-session work, team contexts.

**Skip for:** one-file bugfixes, throwaway scripts, well-scoped work with an obvious path.

## Cheatsheet: typical flows

**New feature, full pipeline:**
```
/ideate my-topic    →  bounded problem map
/spec my-feature    →  spec + decision log
/plan my-feature 1  →  milestone-1.md
/build my-feature 1 →  code, checkpoint commits
/validate           →  type-check + tests
/review my-feature  →  spec compliance report
```

**Clear feature, skip ideation:**
```
/spec my-feature
/plan my-feature
/build my-feature
/review my-feature
```

**Resume after losing context:**
```
/build my-feature 1
  → Finds latest checkpoint commit
  → Offers to resume or rollback
```

**Amend a spec mid-flight:**
```
/spec my-feature
  → "Amend existing spec or start fresh?"
  → Completed milestones preserved; affected ones flagged
```
