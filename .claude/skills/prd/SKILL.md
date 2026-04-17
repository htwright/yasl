---
name: prd
description: Create or update the Product Requirements Document
argument-hint: [amend]
allowed-tools: Read, Glob, Grep, Write, Edit, AskUserQuestion
---

# Product Requirements Document

Create or update `docs/PRD.md` — the single source of truth for what we're building and why.

## If PRD exists and "amend" argument provided:

1. Read the existing PRD
2. Ask what changes are needed
3. Update relevant sections while preserving structure
4. Update the "Last amended" date

## If PRD exists and no "amend" argument:

1. Read the existing PRD
2. Display a summary of current state
3. Ask: **"Update existing PRD, or review current state?"**

## If creating new:

Gather product context through conversation:

1. **Executive summary**: What are we building? (2-3 sentences)
2. **Mission**: Why does this exist? What problem does it solve?
3. **Target users**: Who are the primary users? Secondary?
4. **Core principles**: What values guide decisions?
5. **MVP scope**: What's in v1? What's explicitly deferred?
6. **Success criteria**: How do we measure success? (specific metrics)
7. **Architecture overview**: High-level system design
8. **Technology stack**: What we're using and why
9. **Risks and mitigations**: What could go wrong?

## Output format:

Write to `docs/PRD.md`:

```markdown
# Product Requirements Document

## Executive Summary
[2-3 sentences: what this product is and the core value proposition]

## Mission
[Why this product exists. What problem it solves.]

## Target Users

### Primary
- [User type 1]: [Their goal/need]

### Secondary
- [User type 2]: [Their goal/need]

## Core Principles
1. **[Principle name]**: [Explanation]
2. **[Principle name]**: [Explanation]

## MVP Scope

### In Scope
- [Feature/capability 1]
- [Feature/capability 2]

### Explicitly Deferred
- [What we're NOT building yet and why]

## Success Criteria
| Metric | Target | Measurement |
|--------|--------|-------------|
| [Metric 1] | [Target value] | [How measured] |
| [Metric 2] | [Target value] | [How measured] |

## Architecture Overview
[High-level description of system design, data flow, key components]

## Technology Stack
| Layer | Choice | Rationale |
|-------|--------|-----------|
| [Layer] | [Tech] | [Why] |

## Implementation Phases

### Phase 1: [Name]
- [Goal]
- [Key deliverables]

### Phase 2: [Name]
- [Goal]
- [Key deliverables]

## Risks and Mitigations
| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| [Risk 1] | [H/M/L] | [H/M/L] | [Strategy] |

---
Created: [date]
Last amended: [date]
Status: DRAFT | ACTIVE | COMPLETE
```

## Relationship to other documents:

- **PRD** = WHAT we're building and WHY (product vision, strategy)
- **CLAUDE.md** = HOW we build (conventions, patterns, commands)
- **BACKLOG.md** = WHAT's next (prioritized work items)
- **specs/** = WHAT specifically (detailed feature requirements)

## After writing:

Summarize the PRD and recommend: "PRD created. Use `/spec <feature>` to define specific features from the backlog."
