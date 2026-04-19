# Changelog

## [Unreleased]

### Added
- Initial scaffold: README, config template at `templates/yasl.config.md`, folders for skills, examples, docs.
- MIT license.
- `/start` skill — pipeline-entry triage. Asks 1–3 adaptive questions based on which of the three spaces (problem / solution / implementation) the task lives in, recommends a downstream skill (or "skip the pipeline"), and flags in-progress work via checkpoint commits. Advisory only; does not invoke downstream skills.
- Surfaced `/breakdown` in the public README's pipeline diagram (already existed as a skill, was absent from public docs).

### Next
- Sweep skills clean of project-specific values; read from `.claude/yasl.config.md` instead.
- Write `/setup` skill.
- First worked example in `examples/`.
