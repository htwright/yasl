---
name: breakdown
description: Assess a spec's scope and break it into context-sized milestones
argument-hint: <feature-name>
allowed-tools: Read, Glob, Grep, Write, Edit, AskUserQuestion
---

# Breakdown: Spec → Milestones

Read `docs/specs/$ARGUMENTS.md` and assess whether it can be planned and executed in a single context window.

## Step 1: Read and Analyze

1. Read the spec file
2. Read CLAUDE.md for project conventions
3. Explore relevant existing code to understand scope
4. Identify all the work implied by the spec

## Step 2: Honest Assessment

Provide a **confidence assessment** with this format:

```
## Readiness Assessment

**Spec clarity**: [HIGH | MEDIUM | LOW]
- [What's clear vs ambiguous]

**Scope estimate**: [SMALL | MEDIUM | LARGE | TOO_LARGE]
- Estimated files to touch: [N]
- Estimated complexity: [description]

**Confidence to plan**: [HIGH | MEDIUM | LOW]
- [What gives confidence or causes doubt]

**Open questions that block planning**:
- [List any unresolved questions from spec]

### Recommendation

[One of:]
- ✅ **READY TO PLAN**: Spec is clear, scope fits one context. Run `/plan $ARGUMENTS`.
- ⚠️ **NEEDS BREAKDOWN**: Scope too large. See proposed milestones below.
- 🔙 **RETURN TO SPEC**: Too many open questions. Resolve these first: [list]
```

## Step 3: If breakdown needed

Create `docs/plans/$ARGUMENTS/overview.md`:

```markdown
# $ARGUMENTS - Milestone Overview

**Spec**: ../specs/$ARGUMENTS.md
**Created**: [date]
**Status**: PLANNING

## Milestones

### Milestone 1: [Name]
**Scope**: [What this milestone accomplishes]
**Dependencies**: None
**Estimated size**: [SMALL | MEDIUM]

### Milestone 2: [Name]
**Scope**: [What this milestone accomplishes]
**Dependencies**: Milestone 1
**Estimated size**: [SMALL | MEDIUM]

[Continue as needed...]

## Milestone Sequence

```
M1 → M2 → M3
      ↘ M4 (parallel possible)
```

## Notes
- [Any cross-cutting concerns]
- [Suggested checkpoint boundaries]
```

## Breakdown principles:

1. **Each milestone should fit in one context** (~one session of focused work)
2. **Milestones should be independently testable** when possible
3. **Earlier milestones = foundation** (schema, types, core logic)
4. **Later milestones = integration** (UI, wiring, polish)
5. **Identify parallelizable work** where dependencies allow

## After breakdown:

Recommend: "Run `/plan $ARGUMENTS 1` to create the plan for Milestone 1."
