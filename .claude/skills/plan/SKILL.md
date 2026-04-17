---
name: plan
description: Use when a spec exists at docs/specs/<feature>.md and the user is ready to break it into concrete file-by-file steps. Reads the spec's architecture, contracts, and decision log; produces docs/plans/<feature>/milestone-N.md. Skip if no spec exists — run /spec first.
argument-hint: <feature-name> [milestone-number]
allowed-tools: Read, Glob, Grep, Write, Edit, Bash, AskUserQuestion
---

# /plan — Implementation Decomposition

You take a spec (with architectural approach, components, boundaries, and contracts) and produce an executable plan that `/build` can follow. You bridge "what to build and how it fits together" (spec) and "what to do, in what order, in which files" (plan).

Create a plan for `$ARGUMENTS`.

- Milestone number provided → plan that specific milestone
- No milestone number → assess scope, create milestones if needed, then plan

---

## Upstream Artifacts

Read the spec at `docs/specs/[feature-name].md` — it's your blueprint. If a solution
map exists at `docs/solutions/[feature-name].md`, read that too for architectural context.

If a problem map exists at `docs/ideation/[feature-name].md`, read it for background
but don't plan against it. The spec already translated the problem into a buildable
design. If you find yourself referencing problem-space concerns, check whether the
spec addressed it. If it didn't, that's a signal to return to `/spec`.

Read `.claude/yasl.config.md` for the project's type-check, lint, and test commands — the plan's CHECKPOINT sections reference these (as the names from the config, like `type_check`, not hardcoded command strings).

Read the project's example-pattern files listed in `.claude/yasl.config.md` (route / schema / test patterns) before drafting patterns to follow — they're curated starting points.

---

## Your Roles

### Decomposer (primary)

Break the spec's architecture into ordered, executable steps:

- Each component boundary in the spec suggests a milestone boundary
- Within each milestone, steps follow dependency order (schema → types → implementation → tests)
- Every step names a specific file and a specific action
- Checkpoints align with meaningful integration points, not arbitrary step counts

Think in **dependency chains**, not task lists. The ordering isn't arbitrary — it's
structural. Step 5 depends on step 4 because the type it uses is defined there.

### Pattern Scout

Before writing the plan, do codebase reconnaissance:

- Find existing implementations that match the spec's patterns (start with the example-pattern files listed in `.claude/yasl.config.md`)
- Identify reusable utilities, existing schemas, shared types
- Map the actual file structure to understand where new code fits
- Check conventions in CLAUDE.md and in practice

Produce concrete references: "Follow the pattern in `src/routes/auth.ts` lines
45-80" not "follow existing patterns." The plan should contain enough context that
`/build` rarely needs to explore the codebase itself.

### Risk Assessor

Identify where things are most likely to go wrong:

- Which steps touch shared state or break existing contracts?
- Where are the type-checks likely to fail first?
- What edge cases does the spec's "Open Questions" section flag?
- Where do integration points create coupling risk?

These become your ⚠️ GOTCHA, 🔍 VALIDATE, and 🔒 SECURITY annotations, and they
determine where CHECKPOINT boundaries fall. Checkpoints aren't evenly spaced —
they're at risk boundaries.

---

## Phase 1: Gather Context

1. Read `docs/specs/[feature-name].md` — requirements and architecture
2. Read the spec's **Decision Log** — load-bearing context for step ordering and scope calls. Carry relevant rows into the plan's own Decision Log.
3. Read the spec's **Handoff Notes for /plan** if present — soft signals (preferences, risk intuitions, deployment concerns). Use them to shape milestone boundaries, checkpoint placement, and step ordering. Treat as guidance, not requirements.
4. Read `docs/plans/[feature-name]/overview.md` if it exists — milestones
5. Read `CLAUDE.md` — conventions and patterns
6. Read `.claude/yasl.config.md` — commands to reference in CHECKPOINT sections, example-pattern files
7. Read `docs/solutions/[feature-name].md` if it exists — solution context
8. Read `docs/ideation/[feature-name].md` if it exists — background only

---

## Phase 2: Architecture Decomposition

This phase determines milestone structure. If an `overview.md` already exists from
`/breakdown`, use it. Otherwise, do this work here.

If the spec includes an Architecture section:

1. List the spec's named components and their responsibilities
2. Map each component to a candidate milestone:
   - Component with no dependencies on other new components → early milestone
   - Component that consumes other new components → later milestone
   - Integration between components → dedicated milestone or final phase
3. For each candidate milestone, note:
   - What it receives and produces (from spec contracts)
   - Which existing code it touches (to focus Phase 3 scanning)
4. Present milestone structure to user for validation before proceeding

The spec's contracts tell you where components connect — these are your CHECKPOINT
locations. The spec's tradeoffs tell you what NOT to build — don't plan for excluded
items, and the plan simply shouldn't include them.

If the spec does NOT include an Architecture section, infer structure from the scope
list and dependencies.

---

## Phase 3: Codebase Intelligence

Scan the specific areas each milestone touches, not the whole codebase.

### Pattern Recognition

Search for similar implementations. Start with the example-pattern files in `.claude/yasl.config.md` — those are curated by the repo's maintainer. Then look for adjacent patterns:

```
- If adding a route/endpoint → Find similar routes, note patterns
- If adding a database table → Find similar schema definitions
- If adding a UI component → Find similar components in the same app
- If adding validation → Find existing validation schemas to follow
```

**Document patterns found** with file paths and code snippets.

### Dependency Analysis

Check relevant package manifest files (`package.json`, `pyproject.toml`, `go.mod`, etc.):

- What libraries are available?
- Are there existing utilities that can be reused?
- Any version constraints to be aware of?

### Integration Points

Identify every system that will be touched:

```
- Database: Which tables? New migrations needed?
- API: Which routes? New endpoints?
- Frontend: Which components? State management?
- External: Third-party APIs, webhooks?
```

---

## Backward Flow

If during codebase intelligence you discover the spec doesn't decompose cleanly:

- **Plannable issues** (file paths differ, contracts need minor additions, edge cases
  the spec didn't mention): proceed, note in Gotchas & Risks section.
- **Architectural blockers** (circular dependencies, fundamentally underspecified
  contracts, migration path much harder than assumed): stop planning, set status to
  `BLOCKED`, recommend returning to `/spec` with specifics about what broke.

---

## Consuming Spec Architecture

When the spec includes architecture sections (components, boundaries, contracts),
use them actively to improve plan quality:

### Components → Milestones

The spec's named components (with responsibilities and contracts) map to milestone
boundaries. A component that "receives image URLs and returns product observations"
is a milestone: schema, handler, tests, integration.

### Contracts → Integration Checkpoints

The spec's boundary contracts tell you where components connect. These are your
CHECKPOINT locations — the moments where you verify that component A's output
actually works as component B's input.

### Tradeoffs Accepted → Scope Guards

The spec's tradeoffs are your "do NOT build this" list. Include explicit
"NOT IN SCOPE" notes at the steps where scope creep is most tempting.

### Migration Path → Sequencing

If the spec includes a migration path, it directly informs your step ordering.
"Introduce new alongside existing, then migrate callers, then remove old"
is a three-phase plan structure handed to you by the spec.

---

## Detail Level Calibration

**Under-planned** — steps are vague ("implement the handler"), file paths missing,
no pattern references, `/build` will have to re-explore the codebase.

**Over-planned** — steps specify implementation details the engineer should decide,
plan is longer than the code it describes, annotations on every step (risk dilution),
steps that are really sub-steps of a single action.

**Right level** — every step is specific enough to execute without ambiguity, but not
so detailed that it removes judgment from the implementer.

---

## Phase 4: Write the Plan

Write to `docs/plans/[feature-name]/milestone-N.md` (or `plan.md` if no milestones):

````markdown
---
feature: [feature-name]
milestone: [N]
title: [Milestone title]
status: NOT_STARTED
created: [date]
spec: ../../specs/[feature-name].md
---

## Upstream Context

- **Spec**: ../../specs/[feature-name].md
- **Solution Map**: ../../solutions/[feature-name].md (if exists)
- **Problem Map**: ../../ideation/[feature-name].md (if exists)
- **Architecture Approach**: [Name from spec's chosen approach, or "N/A"]

# Milestone N: [Title]

## Goal

[One sentence: what this milestone accomplishes]

## Prerequisites

- [ ] [Anything that must be true before starting]

## Patterns to Follow

### [Pattern Category]

From `path/to/example.ts`:

```typescript
// Key code snippet showing the pattern
```
````

**Apply to**: [Where this pattern applies in our implementation]

## Files to Touch

- `path/to/file.ts` - [what changes]
- `path/to/other.ts` - [what changes]

## Steps

### Phase 1: [Name, e.g., "Schema Changes"]

- [ ] 1. [Specific action with file path]
- [ ] 2. [Specific action with file path]
- [ ] 3. [Specific action with file path] ⚠️ GOTCHA: [edge case to watch]

#### CHECKPOINT

**Run:**

- `type_check` (from `.claude/yasl.config.md`)
- [any step-specific check]

**Verify:**

- [What to check manually if needed]

**Commit recommendation:** `[type]: [description]`

---

### Phase 2: [Name, e.g., "API Implementation"]

- [ ] 4. [Specific action]
- [ ] 5. [Specific action] 🔍 VALIDATE: [thing to verify works]

#### CHECKPOINT

**Run:**

- `type_check` (from `.claude/yasl.config.md`)
- `test` (from `.claude/yasl.config.md`)

**Verify:**

- [What to check]

**Commit recommendation:** `[type]: [description]`

---

## Completion Criteria

- [ ] All steps checked off
- [ ] All checkpoints pass
- [ ] [Spec success criteria this satisfies]

## Gotchas & Risks

- ⚠️ [Edge case or risk identified during planning]
- ⚠️ [Another potential issue]

## Decision Log

Carry forward any spec decision whose **Revisit If…** condition could plausibly
trigger during this milestone's steps — skip the rest. Add new planning decisions
(step ordering rationale, pattern choices, scope boundary calls).

| #   | Decision                | Source   | Rationale                         | Revisit If… |
| --- | ----------------------- | -------- | --------------------------------- | ----------- |
| S1  | [From spec]             | Spec #1  | [Why, in planning context]        | [Condition] |
| P1  | [New planning decision] | Planning | [Why this ordering/pattern/scope] | [Condition] |

`/build` consults this table when about to deviate from the plan.

## Notes

- [Patterns identified during analysis]
- [Decisions made during planning]

```

---

## Annotations

Use these inline annotations in steps:

| Annotation | Meaning |
|------------|---------|
| ⚠️ GOTCHA | Edge case or risk to watch for |
| 🔍 VALIDATE | Something to verify works after implementation |
| 🔒 SECURITY | Security-sensitive step requiring extra care |
| 📊 PERF | Performance-sensitive step |

---

## Plan-quality bar

- Every step names a specific file and a specific action.
- Pattern references include file path and line range — not "follow existing patterns."
- Checkpoints fall at risk boundaries (contract integrations, shared-state changes, migration steps), not at fixed intervals.
- Annotations (⚠️ 🔍 🔒) appear only where the risk is real. Annotating every step dilutes them.
- Steps are specific enough to execute without ambiguity, loose enough to leave naming and internal structure to `/build`.

## Out of scope for this skill

- Writing implementation code — `/build` does that
- Revisiting architectural decisions — `/spec` does that
- Planning for items the spec explicitly excluded

---

## After Creating Plan

1. **Write Handoff Notes** — if there's context from this session that won't survive into `/build` and isn't already captured by the plan steps, append a `## Handoff Notes for /build` section. Up to 5 bullets. Skip the section entirely if nothing fits — fake bullets are worse than none. Examples of context worth recording:
   - "Steps 3-5 are the highest-risk sequence — if anything breaks, it'll be there"
   - "The pattern in `[example-file from config]` is the closest match but has a quirk at line 80"
   - "User wants incremental deployability — each phase should leave the system working"
   - "Checkpoint 2 is where we'll know if the migration path actually works"

2. **Present the plan summary** with the file path, then ask the user to confirm or flag adjustments before `/build`. One question, not a checklist. If they confirm, recommend: "Run `/build [feature-name] [milestone]` to execute."
```
