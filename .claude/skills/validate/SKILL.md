---
name: validate
description: Run automated validation (type-check, lint, tests) using the commands defined in .claude/yasl.config.md. Separate from /review, which checks spec compliance.
argument-hint: [scope]
allowed-tools: Bash, Read
---

# Automated Validation

Run your project's automated checks. This is separate from `/review`, which checks against spec criteria.

## Startup

1. Read `.claude/yasl.config.md`. If it's missing, stop and tell the user: *"YASL isn't configured yet — run `/setup` first."*
2. From the `## Commands` section, note which of the following are configured (a value of `(not configured)` or a missing line counts as unconfigured):
   - `type_check` (required)
   - `lint` (recommended)
   - `test` (required)
   - `test_secondary` (optional)
   - `test_production` (optional)

All command executions below use the strings from `yasl.config.md` — never hardcode them.

## Scope Options

- _(no argument)_: Run all configured validations **except** production
- `primary`: Primary test suite only
- `secondary`: Secondary test suite only (skip if not configured)
- `production`: Production smoke tests (skip if not configured)
- `quick`: Type-check only — fastest feedback loop

## Validation Order

Always runs type-check first. Lint before tests. Production last, and only when explicitly requested.

| Scope      | Type-Check | Lint | Primary Test | Secondary Test | Production |
| ---------- | ---------- | ---- | ------------ | -------------- | ---------- |
| _(none)_   | Yes        | Yes  | Yes          | Yes (if conf.) | No         |
| primary    | Yes        | Yes  | Yes          | No             | No         |
| secondary  | Yes        | Yes  | No           | Yes            | No         |
| production | Yes        | Yes  | No           | No             | Yes        |
| quick      | Yes        | No   | No           | No             | No         |

## Execution

For each suite in scope:

1. Run the configured command.
2. Capture stdout/stderr and exit code.
3. On failure, **continue to the next suite** — the goal is a complete report, not a fast bail. Mark the overall result as FAIL.
4. Skip any suite whose command is not configured.

## Output Format

```markdown
# Validation Report

## Summary

| Suite          | Status         | Duration |
| -------------- | -------------- | -------- |
| Type Check     | PASS/FAIL      | Xs       |
| Lint           | PASS/FAIL/SKIP | Xs       |
| Primary Test   | PASS/FAIL      | Xs       |
| Secondary Test | PASS/FAIL/SKIP | Xs       |
| Production     | PASS/FAIL/SKIP | Xs       |

## Overall: PASS / FAIL

## Details

### Type Check
[Output summary or "All clear"]

### Primary Test
- Total: X tests
- Passed: X
- Failed: X
  [Failure details if any]

[...]

## Failures Requiring Attention

1. [Specific failure with file:line if available]

---

Validated: [timestamp]
```

## Failure Handling

If validations fail:

1. Report the specific failures with file paths and line numbers.
2. Do NOT attempt to fix automatically — `/validate` is diagnostic, not corrective.
3. Recommend: *"Fix the issues above and run `/validate` again."*

## Relationship to /review

| `/validate`          | `/review`            |
| -------------------- | -------------------- |
| Automated checks     | Spec compliance      |
| Type safety          | Feature completeness |
| Test suite           | Success criteria     |
| Machine verification | Human verification   |

Run `/validate` frequently during `/build`. Run `/review` when a feature is complete.

## Common Issues

### Type check failures

- Usually caught first, fix before running tests.
- Check imports, return types, null handling.

### Test failures

- Check that the database schema matches what tests expect.
- Verify environment variables are set.
- Check for port conflicts if tests start a server.

## Out of scope for this skill

- Fixing the failures it finds — `/validate` reports; the human or `/build` fixes.
- Running commands not listed in `yasl.config.md`. If a check is missing, add it to the config.
- Deciding what "quality" means — the config defines the surface; this skill just runs it.