---
name: prime
description: Load project context before planning complex features
argument-hint: [feature-name]
allowed-tools: Read, Glob, Grep, Bash
---

# Prime Context

Load comprehensive project context to prepare for planning. Use before `/plan` on complex features.

## Purpose

Deep context loading enables better plans by:
- Understanding existing patterns before proposing new code
- Identifying integration points that might be missed
- Recognizing conventions that should be followed
- Surfacing potential conflicts or dependencies

## Execution

### Phase 1: Core Documents

Read these essential files:
1. `CLAUDE.md` — project conventions, patterns, decisions
2. `.claude/yasl.config.md` — commands and example-pattern files for this repo
3. `docs/PRD.md` — product requirements and vision (if it exists)
4. `docs/BACKLOG.md` — current priorities and context (if it exists)

### Phase 2: Feature Context (if feature-name provided)

1. Read `docs/specs/$ARGUMENTS.md` if it exists
2. Read `docs/plans/$ARGUMENTS/` contents if they exist
3. Search for related code:
   - Grep for feature name in codebase
   - Identify files that will likely be touched

### Phase 3: Codebase Analysis

1. **Structure scan**: Map out the project structure. Start with the example-pattern files listed in `.claude/yasl.config.md` — these are the repo's curated reference points. From each, walk outward to understand the shape of the surrounding directory. If the project is a monorepo, scan each workspace's top-level `src/` (or equivalent).

2. **Pattern recognition**: Find implementations similar to the feature's shape.
   - Search for patterns matching the feature type (routes, schemas, components, validation)
   - Note coding conventions used in practice, not just those in `CLAUDE.md`

3. **Dependency analysis**: Check relevant package manifests (`package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`, etc.):
   - What libraries are available?
   - What versions are in use?

### Phase 4: Recent Changes

1. Check recent git history (last 10 commits)
   ```bash
   git log --oneline -10
   ```

2. If feature-related files exist, check their recent changes
   ```bash
   git log --oneline -5 -- [relevant-paths]
   ```

## Output Format

Present a context summary:

```markdown
# Context Summary: [Feature Name or "General"]

## Project State
- **Current focus**: [from BACKLOG.md or recent commits]
- **Recent work**: [from git history]

## Relevant Patterns

### [Pattern Category 1]
- Location: `path/to/example.ts` (from yasl.config.md's example_patterns)
- Key insight: [What to follow]

### [Pattern Category 2]
- Location: `path/to/example.ts`
- Key insight: [What to follow]

## Files Likely Affected
- `path/to/file.ts` — [Why]
- `path/to/other.ts` — [Why]

## Integration Points
- [System/feature 1]: [How it connects]
- [System/feature 2]: [How it connects]

## Conventions to Follow
1. [Convention from CLAUDE.md]
2. [Convention from CLAUDE.md]

## Potential Concerns
- [Risk or consideration 1]
- [Risk or consideration 2]

---
Ready for: `/plan [feature-name]`
```

## When to Use

- Before `/plan` on features touching multiple systems
- When starting work after a break from the project
- When onboarding to understand the codebase
- Before making architectural decisions

## After priming

Recommend: "Context loaded. Run `/plan $ARGUMENTS` to create an implementation plan."

## Out of scope for this skill

- Writing specs or plans — priming loads context, nothing more
- Running tests or build commands — `/validate` does that
