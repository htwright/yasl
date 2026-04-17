---
name: commit
description: Stage, commit, and push changes with a conventional commit message. Reads commit_scopes from .claude/yasl.config.md.
argument-hint: [message] [--no-push] [--all]
allowed-tools: Bash, Read
---

# Commit Changes

Stage, commit, and push the current changes. Generates a conventional commit message based on the diff.

## Startup

Read `.claude/yasl.config.md` for the project's `Commit Scopes` list. If the file is missing or the list is empty, you can still proceed — just ask the user for an appropriate scope or omit the scope if the project doesn't use them.

## Arguments

- No argument: Auto-generate commit message from diff, commit, and push
- `<message>`: Use the provided message as the commit summary
- `--no-push`: Skip pushing to remote after committing
- `--all`: Include all modified tracked files (skip file selection)

Examples:

```
/commit
/commit --no-push
/commit fix auth redirect
/commit --all
/commit fix auth redirect --no-push
```

## Procedure

### 1. Gather context (run in parallel)

```bash
git status
git diff --staged
git diff
git log --oneline -5
```

### 2. Determine what to stage

- If there are already staged changes and no argument `--all`, commit only what's staged
- If nothing is staged, identify which modified/untracked files relate to a single logical change
- **Never stage** files that look like secrets (`.env`, credentials, tokens)
- **Never stage** `nul` (Windows artifact)
- If unrelated changes exist, ask the user which files to include
- With `--all`: stage all modified tracked files (still skip secrets and `nul`)

### 3. Draft the commit message

Follow the project's conventional commit style visible in `git log`:

```
<type>(<scope>): <short summary>
```

Types: `feat`, `fix`, `refactor`, `test`, `docs`, `chore`, `style`, `perf`

**Scope:** pick from the `Commit Scopes` list in `.claude/yasl.config.md`. If none of the configured scopes fit, ask the user to either (a) pick one anyway, (b) add a new scope to the config, or (c) omit the scope for this commit.

Rules:

- Summary line under 72 characters
- Focus on **why**, not what
- If the user provided a message, use it as the summary (still add type/scope prefix if missing)
- Add a blank line + body only if the change is non-obvious
- Always end with Claude Code's standard co-author trailer (the runtime fills in the current model version)

### 4. Commit

```bash
git add <files>
git commit -m "$(cat <<'EOF'
<message>

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

### 5. Verify

```bash
git status
```

Confirm the commit succeeded and report the short SHA.

### 6. Push (default — skip only if `--no-push` was specified)

```bash
git push
```

Report the result. If the push fails (e.g., behind remote), report the error — do NOT force-push.

## Important

- Use `git add` with explicit file paths — never `git add -A` or `git add .`
- Never amend a previous commit unless the user explicitly says "amend"
- Never force-push
- Never skip hooks (`--no-verify`)
- If pre-commit hooks fail, report the failure — do not retry automatically
- Pass commit messages via HEREDOC to preserve formatting

## Out of scope for this skill

- Resolving merge conflicts
- Rebasing or interactive history editing
- Creating branches or PRs (use `gh pr create` separately)
