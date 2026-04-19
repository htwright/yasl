# Vision

YASL turns thought into code without losing the thought.

## The problem YASL exists to solve

AI coding works well when the problem is small, the path is obvious, and one session is enough. It falls apart when any of those break — when the problem is fuzzy, when the path has choices, when the work spans days.

The failure mode isn't the AI. It's the loss of context between the thinking that shaped a decision and the code that carried it out. Something was understood at 10am, a detail was reasoned through at noon, a tradeoff was accepted at 2pm — and by the time code is being written, the tradeoff is invisible. The AI chooses something different, not because it's worse but because it can't see the reason. The human reviews the code, can't reconstruct what was decided and what was assumed, and either accepts a quiet re-architecting or starts over.

YASL's purpose is to keep reasoning alive from the first vague idea to the final commit.

## The three spaces

Work moves through three spaces, and each one asks a different question:

- **Problem space** — _is this the right problem to solve?_ The problem is still fuzzy. The edges haven't been found. Constraints and tensions are emerging but not settled.
- **Solution space** — _given the problem, what's the right approach?_ The problem is bounded. Multiple architectures could work. The question is which tradeoffs to accept.
- **Implementation space** — _given the approach, what's the sequence of moves?_ The architecture is chosen. The work is mechanical: decompose into steps, execute each one, validate.

Each space has its own discipline. In problem space you do not propose solutions. In solution space you do not write code. In implementation space you do not re-architect. When these disciplines blur — when a spec drifts back into problem space, or a build silently changes the architecture — the reasoning chain breaks.

YASL's skills are shaped by which space they live in:

- `/ideate` lives in problem space. It cannot suggest solutions. It refuses to draft architectures.
- `/spec` lives in solution space. It treats the bounded problem as fixed. When the user tries to reopen problem-space questions, it names this and suggests returning to `/ideate`.
- `/plan` and `/build` live in implementation space. When the plan reveals a flaw in the spec, `/build` stops — it does not re-architect silently.

This is the core discipline. Every other mechanism in YASL — handoff notes, decision logs, backward flow, checkpoint commits — is in service of it.

## Why artifacts on disk

Memory is the wrong place for a reasoning chain. Sessions end. Contexts fill. The AI forgets, and so does the human. YASL writes the reasoning to disk in structured artifacts — problem maps, solution maps, specs, plans — so that the next session can pick it up without reconstruction.

This is what makes "it took us an afternoon to reason through this" survive to the moment when the code is actually written. The reasoning is not in the skill. It's in the artifact the skill produced.

Decision logs and handoff notes exist for this reason. A decision log answers "why did we choose this?" three weeks later, when someone (possibly you) is tempted to change it. Handoff notes answer "what did the last phase understand that the formal artifact doesn't capture?" — the emotional weight of a choice, the thing that came up four times unprompted, the pattern concern that matters for the plan even though it wasn't part of the spec.

## Scope calibration

A separate discipline, equally load-bearing: **the quality of a task's specification determines how disciplined the agent's execution will be.**

Under-specified tasks invite ambition. An agent given one sentence will do what seems useful — which often means more than asked. This is not a defect of the agent; it is the logical response to a vague instruction. The same agent, given a well-specified task with explicit scope constraints, will stay tight.

This has a consequence for when to use YASL at all:

- When the work is obvious, small, and the path is clear, running the full pipeline is overhead. Go straight to implementation. The thinking has already happened.
- When the work is ambiguous, multi-file, or architecturally load-bearing, skipping the pipeline means paying the cost later — in scope drift, in silent re-architecting, in code that doesn't match what was meant.

YASL is not a tax on all work. It is a tax on thinking that would otherwise be lost. When there is no thinking to preserve, there is nothing to tax.

## What YASL is not

- **Not a thinking substitute.** YASL is a thought enhancer, not a thought replacer. If you can see the work clearly in your head, YASL helps you get an AI to do it well. If you can't — if the shape is fuzzy in your own mind — no amount of prompting or pipeline machinery will produce work that holds up. The distance between "AI as thought enhancement" and "AI as thought replacement" is the same as the distance between radar-aware cruise control and full self-driving: the first is real, useful, and deployed; the second is an aspiration that's 80% there and has been for years. YASL is built for the first. `/ideate` asks questions, it does not answer them. `/spec` surfaces tradeoffs, it does not resolve them. The human decides.
- **Not a productivity layer.** YASL adds friction on purpose where friction is load-bearing. The goal is better work, not more work per unit time.
- **Not a process to follow.** YASL is a toolset for when the work calls for it. Using it for everything is as wrong as using it for nothing.
- **Not opinionated about stack.** Skill bodies read project-specific values from `.claude/yasl.config.md`. The pipeline is a shape, not a set of languages or frameworks.

## Where this points

Two directions that aren't implemented but follow from the above:

**Triage at the entry point.** A user with a task in hand needs to know which space it belongs in before they know which skill to run. The choice between `/ideate`, `/spec`, and "skip the pipeline" is itself a decision worth supporting — and it is exactly the decision where miscalibration is most costly.

**Mode-aware artifacts.** The distinction between "I'm thinking out loud" and "I'm committing to this" is real and valuable, but YASL's current artifacts don't carry it. A spec that is still provisional reads the same as a spec that is ready to build against. There is room for artifacts that know which mode they are in, so that downstream skills can treat them appropriately.

Neither of these is promised. They are the shape of what YASL could become if it stays true to the three-space discipline and the scope-calibration principle.

---

_This document describes what YASL is. For what it does, see the [README](README.md). For how to build with it, see [`.claude/skills/README.md`](.claude/skills/README.md)._