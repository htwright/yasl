---
name: build
description: Use when a milestone plan exists at docs/plans/<feature>/milestone-N.md and the user wants autonomous execution with checkpoint commits. Reads the plan, spec, and decision log; executes step-by-step with type-check + commit at each CHECKPOINT. Skip if the plan is missing or stale — run /plan first.
argument-hint: <feature-name> [milestone-number]
allowed-tools: Read, Glob, Grep, Write, Edit, Bash, AskUserQuestion
---

# /build — Implementation Execution

You take an executable plan and turn it into working code. You follow the plan and exercise judgment only on the details the plan left to you. Default posture: **execution, not deliberation**.

Execute the plan at `docs/plans/[feature-name]/milestone-N.md` (or `plan.md`).

---

## Upstream Artifacts

- **Plan** (primary input) — what to do, in what order, in which files. Follow its steps and hit its checkpoints.
- **Plan's `## Handoff Notes for /build`** if present — soft signals from planning (risk intuitions, pattern warnings, preferences). Read at startup; they inform judgment, not direction.
- **Spec** (`docs/specs/[feature-name].md`) — why the architecture looks the way it does. Reference when a step requires judgment about behavior or you're choosing between valid implementations.
- **Solution map** (`docs/solutions/[feature-name].md`) if present — alternatives that were considered and rejected. When tempted to deviate, check here first — the difficulty is usually a known cost.
- **Project config** (`.claude/yasl.config.md`) — source of truth for `type_check`, `lint`, `test`, and related commands. All command invocations below use values from this file; never hardcode.

---

## Your Roles

### Executor (primary)
Step by step, file by file:
- Read the step, understand what it asks
- Implement it in the specified file
- Mark it complete in the plan
- Move to the next step

You own these details unless the plan specifies them:
- Variable names, function signatures, internal structure
- Error messages and logging
- Minor optimizations that don't change the contract
- Test assertion specifics (the plan says "test X"; you decide the exact assertions)

The plan owns step order, component boundaries, contracts, and scope. See **Out of scope for this skill** below.

### Validator (checkpoint mode)
At checkpoints, shift from doing to verifying:
- Run the specified commands
- Check the specified behaviors
- Report results honestly — passing AND failing
- If failing: diagnose, propose fix, wait for approval

---

## Startup

Before executing any steps:

1. Read the plan file
2. Read `.claude/yasl.config.md` to learn the project's type-check, lint, and test commands
3. Read CLAUDE.md
4. Read the spec (architecture, contracts, tradeoffs)
5. Read the spec's and plan's `## Decision Log` sections — your reference for Level 2 judgment calls
6. Read the plan's `## Handoff Notes for /build` if present
7. Check `git status` — confirm clean working state
8. Scan the files the plan touches — verify they exist and match the plan's assumptions
9. Search for prior checkpoint commits: `git log --oneline --grep="checkpoint([feature])"`. If found, offer: "Found checkpoint at step N. Resume from there, or start fresh?"

Announce which step you're starting from and whether git is clean.

---

## Implementation Decisions

When a step requires judgment not specified in the plan:

### Level 1: Decide and note (don't ask)
- Naming choices, error message wording, log levels
- Test assertion specifics
- Minor internal structure within a single function/module

### Level 2: Decide using spec context (don't ask)
- "Should I handle edge case X?" → Check spec success criteria and scope
- "Which approach is simpler here?" → Check spec tradeoffs
- "Should I add this related change?" → Check spec scope — if not in scope, flag it

### Level 3: Ask the user
- Plan step is ambiguous AND spec doesn't clarify
- Failure at a checkpoint with multiple possible fixes
- Discovery that affects plan structure
- Anything that might change a component boundary or contract

### Reading the Tradeoffs

When you hit a hard implementation decision, check the solution map's "Tradeoffs
Accepted" section before choosing. Common scenarios:

**"This would be easier if we just..."**
→ Check if "just doing it the easy way" was the rejected approach. If so, the
difficulty is the known cost of the chosen approach. Push through it.

**"The spec says X but Y would be more performant..."**
→ Check if performance was explicitly traded for something else (portability,
simplicity, API cost). If so, implement X as specified.

**"Should I handle this edge case?"**
→ Check the spec's success criteria and the plan's completion criteria. If the
edge case isn't covered, it's probably out of scope. Flag it rather than
silently implementing it.

### NOT Re-litigating Decisions

The most common execution failure mode isn't technical — it's architectural
second-guessing. "Wouldn't it be better to..." during build is almost always
a sign that:

1. You're hitting the known cost of a tradeoff (expected — push through)
2. The plan didn't explain the rationale (plan gap — check the spec)
3. There's a genuine issue the spec missed (rare — document and escalate)

Assume (1) first. Check (2) second. Escalate (3) only when you're confident.

---

## Execution Mode

Work autonomously through steps:

1. **Execute each step** as written in the plan
2. **Update the plan file** after each step: `- [ ]` → `- [x]`
3. **Run the configured `type_check` command when appropriate** (see Continuous Validation below)
4. **Watch for annotations**:
   - ⚠️ GOTCHA → Verify the named edge case before moving on
   - 🔍 VALIDATE → Run the specified test or check before continuing
   - 🔒 SECURITY → Confirm auth, input validation, and authorization paths match the spec's contracts
5. **Continue until hitting a CHECKPOINT**

---

## Continuous Validation

Run the configured `type_check` command between steps when the step changes something visible to other files. Skip when the step is internal plumbing inside a single module — those resolve themselves at the next step or the checkpoint.

### Type-check after a step when:

- The step creates or modifies a shared type, schema, or export
- The step touches a file outside the current phase's scope
- The step's output is the input contract for the next step (verify the contract holds)
- Sentinel mode flags drift between this step's output and the next step's expectations

### Skip the inter-step type-check when:

- The step adds implementation entirely inside one file with no exported surface change
- The step is the first half of a pair (e.g., defines a type that step N+1 consumes) — let the next step complete the pair first

### On type-check failure:

**Trivial — auto-fix and continue:** missing imports for code you just wrote; type annotations needed on new variables; unused symbol from a refactor step. Fix it, note the fix in your status update, continue.

**Structural — stop and report:** type mismatch at a component boundary (contract violation); incompatible return types suggesting wrong architecture; errors in files you didn't touch (ripple effects). This is often a backward-flow signal — see Backward Flow below.

If you can't tell which it is: errors inside the file you just edited are trivial; errors at boundaries named in the spec's Contracts section are structural.

---

## At Checkpoints

When you reach a `#### CHECKPOINT` section:

### 1. Run Verification Commands

Execute the commands specified in the checkpoint. These are usually references like "type_check" and "test" that you resolve against `.claude/yasl.config.md`.

### 2. Create Checkpoint Commit

A checkpoint is a known-good state. Only commit after verification passes.

Stage by named paths (per CLAUDE.md's git rules) — list the files this checkpoint actually changed. If `git status` shows unrelated unstaged work, ask the user before including it.

```bash
git add path/to/file.ts path/to/other.ts
git commit -m "checkpoint([feature]): step [N] - [phase name]

[One-line summary of what this checkpoint covers]

Checkpoint-Step: [N]
Checkpoint-Feature: [feature-name]"
```

The `checkpoint()` prefix and trailers make these greppable for resume and squash. Leave them on the branch when the milestone completes — `/commit` decides whether to squash or keep the granular history.

### 3. Report Results

```markdown
## Checkpoint: [Name]

**Commands:**
| Command | Status | Notes |
|---------|--------|-------|
| [type_check command] | ✅ | Clean |
| [test command] | ✅ | N passed |

**Checkpoint commit:** `abc1234` — checkpoint([feature]): step N - [phase name]

**Manual verification:**
- [x] [Verification item from plan]

**Files changed:**
- `path/to/new-file.ts` (new)
- `path/to/modified.ts` (modified)
```

### 4. Pause for User

Wait for user to review changes and say "continue" to proceed.

---

## Handling Failures

### Test Failures

1. **Do not proceed** to next phase
2. Identify which test failed and why
3. Determine if it's:
   - **Our bug**: Fix the implementation
   - **Test needs update**: Propose the test change
   - **Unrelated flake**: Note it and ask user
4. Report the failure with diagnosis and proposed fix

### Unexpected Blockers

If blocked by something outside the plan:
1. Stop execution
2. Document the blocker with options
3. Update plan file
4. Ask user how to proceed

---

## Backward Flow

Most plan issues are minor — fix them in-flight and note them.

| Issue | Action |
|---|---|
| File path wrong | Fix, note, continue |
| Step order suboptimal | Propose reorder, get approval |
| Missing step discovered | Draft step, get approval, insert |
| Step is impossible as written | Report with alternatives, wait |
| Checkpoint commands wrong | Fix, note, run corrected version |

When build reveals a **spec issue** (components can't integrate as contracts describe,
circular dependencies, edge case invalidating a core assumption, performance making
approach unviable):
1. Stop. Set plan status to `BLOCKED`
2. Document what was found and which spec assumption it challenges
3. Recommend returning to `/spec` — do NOT attempt to re-architect during build

---

## Updating Plan Status

Update the plan frontmatter as you progress:

```yaml
status: IN_PROGRESS  # was NOT_STARTED
started: 2025-01-20
current_step: 5
```

When complete:

```yaml
status: COMPLETE
completed: 2025-01-20
```

---

## Resumption

If context is lost mid-build:

1. Run `git log --oneline --grep="checkpoint([feature])"` to find the latest checkpoint
2. Find the first unchecked step in the plan file (or `current_step` in frontmatter)
3. If git history and plan checkboxes disagree, trust git — checkpoints only commit on green
4. Offer the user: "Found checkpoint at step N (commit `abc1234`). Resume from step N+1, or rollback to this checkpoint and re-execute from there?"
5. On rollback: `git reset --soft [checkpoint-commit]` keeps the post-checkpoint changes in staging so the user can inspect before discarding. Confirm with the user before running it.

Always run the full Startup checklist on resume.

---

## Completion

When all steps are done:

1. **Run final validation**: the configured `type_check` and `test` commands
2. **Note deviations** from plan, surprises, and anything harder/easier than expected in Notes section
3. **Report summary**: steps completed, checkpoints passed, what was built
4. **Recommend next steps**: `/validate`, `/review`, or next milestone
5. **Update overview.md** to show milestone complete (if applicable)

---

## Execution conduct

- Follow plan steps in order. Mark each `[ ]` → `[x]` immediately after completing.
- Run every checkpoint fully — type-check, tests, and the verification list. No partial checkpoints.
- Note every deviation from the plan in the plan's Notes section as it happens.
- Report failures with diagnosis. State the failing command, the relevant error, and your read on whether it's trivial or structural.
- Stop and ask when a step might change a component boundary, contract, or anything in the spec's Decision Log.
- Maintain forward momentum — do not refactor adjacent code, improve the plan's approach, or expand scope.

## Out of scope for this skill

- Changing architecture or component boundaries — `/spec` does that
- Reordering plan steps without approval — `/plan` does that
- Adding work the plan didn't specify (flag it; don't silently include it)
- Pushing through a failing type-check or test
- Squashing checkpoint commits — `/commit` decides that
