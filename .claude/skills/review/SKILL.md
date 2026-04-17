---
name: review
description: Review completed work against the original spec and project conventions
argument-hint: <feature-name>
allowed-tools: Read, Glob, Grep, Bash, AskUserQuestion
---

# Review: Validate Against Spec

Review the work done for `$ARGUMENTS` against the original specification.

---

## Part 1: Gather Artifacts

1. Read `docs/specs/$ARGUMENTS.md` — the requirements
2. Read `docs/plans/$ARGUMENTS/overview.md` — the milestone breakdown (if it exists)
3. Read all milestone plans — the implementation record
4. Read `CLAUDE.md` — project conventions
5. Read `.claude/yasl.config.md` — the commands `/review` will run

---

## Part 2: Spec Compliance

For each **Success Criterion** in the spec:

```markdown
### Criterion: [description]

**Status**: ✅ COMPLETE | ⚠️ PARTIAL | ❌ NOT DONE

**Evidence**:
- [File/test/behavior that proves this is done]

**Notes**:
- [Any caveats or follow-up needed]
```

---

## Part 3: Code Quality Review

Review the new/modified code for quality issues.

### 3.1 Convention Compliance

Cross-check against `CLAUDE.md`'s conventions section (the source of truth for this project's style). Common items to look for — apply only those your `CLAUDE.md` actually prescribes:

- [ ] Type system: strict mode honored, no escapes (`any`, `# type: ignore`, unchecked casts)
- [ ] Error handling follows project pattern (Result types, exceptions, error returns — match `CLAUDE.md`)
- [ ] Data conventions from `CLAUDE.md` (ID format, money representation, timestamp format, etc.)
- [ ] Validation at system boundaries (input parsers, schema checks)
- [ ] Consistent error response / return format across the surface changed

If a convention in `CLAUDE.md` conflicts with what the spec prescribed, flag the mismatch — don't silently pick one.

### 3.2 Code Cleanliness

- [ ] No commented-out code left behind
- [ ] No debug logging statements (`console.log`, `print`, `fmt.Println` for debugging)
- [ ] No TODO comments without a tracking reference
- [ ] No unused imports or variables
- [ ] Consistent naming conventions
- [ ] Functions are reasonably sized (single responsibility)

### 3.3 Security Checklist (OWASP-inspired)

For any user-facing changes:

- [ ] **Injection**: All user input validated/escaped (SQL, command, XSS)
- [ ] **Auth**: Protected routes require authentication
- [ ] **Authorization**: Users can only access their own data
- [ ] **Sensitive Data**: No secrets in code; env vars used correctly
- [ ] **Rate Limiting**: Public endpoints have reasonable limits (if applicable)
- [ ] **Error Messages**: No stack traces or internal details exposed to users
- [ ] **CORS**: Appropriate origin restrictions (if applicable)

Mark N/A if not applicable to this feature.

### 3.4 API Endpoint Review (if applicable)

For each new/modified endpoint:

| Endpoint | Auth | Validation | Errors | Tests |
|----------|------|------------|--------|-------|
| `[METHOD] /path` | ✅ | ✅ | ✅ | ✅ |

- **Auth**: Requires appropriate authentication
- **Validation**: Request body validated against a schema
- **Errors**: Uses the project's error-response helpers
- **Tests**: Covered by smoke / integration / security tests as appropriate

---

## Part 4: Test Verification

Run the commands from `.claude/yasl.config.md`:

- `type_check` — always
- `lint` — if configured
- `test` — always
- `test_secondary` — if configured and the secondary scope was touched

Skip `test_production` unless the user explicitly asks for it — `/review` should not hit production by default.

Report results with any failures highlighted.

---

## Part 5: Summary Report

```markdown
# Review Summary: $ARGUMENTS

## Spec Completion: [X/Y criteria met]

| Criterion | Status | Notes |
|-----------|--------|-------|
| [name] | ✅ | [notes] |
| [name] | ⚠️ | [what's missing] |

## Code Quality

### Conventions: [PASS | X issues found]
- [List any issues]

### Security: [PASS | X concerns]
- [List any concerns]

### Endpoints: [N endpoints reviewed]
- [Summary]

## Test Results: [PASS | FAIL]

| Suite | Status | Count |
|-------|--------|-------|
| Type check | ✅ | - |
| Primary tests | ✅ | N passed |
| Secondary tests | ✅ | N passed |

## Recommendation

[One of:]

✅ **READY TO MERGE**
All criteria met, code quality good, tests passing.

⚠️ **NEEDS WORK**
Address these items before merging:
1. [Specific issue]
2. [Specific issue]

🔒 **SECURITY CONCERN**
Review required for:
1. [Security issue]

🔙 **SPEC MISMATCH**
Implementation diverged from spec. Discussion needed:
1. [Divergence]
```

---

## If Issues Found

Ask user: "Would you like me to create a follow-up plan to address these gaps?"

For minor issues, offer to fix them directly.

---

## Quick Reference: What Each Check Catches

| Check | Catches |
|-------|---------|
| Spec compliance | Missing features, incomplete work |
| Conventions | Inconsistent code, future maintenance issues |
| Security | Vulnerabilities before they ship |
| Code cleanliness | Technical debt, debugging artifacts |
| Tests | Regressions, integration issues |

## Out of scope for this skill

- Running production tests by default — explicit opt-in only
- Enforcing conventions not documented in `CLAUDE.md` (the human author owns the convention list)
- Rewriting architecture — a spec mismatch flags, but doesn't re-architect
