---
name: start
description: Use when you have a task in mind but aren't sure which skill to run — or when returning to work after a break. Asks 1-3 calibrating questions, then recommends a pipeline entry point (or "skip the pipeline") based on which space the work belongs in. Also detects in-progress work via checkpoint commits. Does not invoke downstream skills — the user chooses whether to proceed.
argument-hint: [brief task description]
allowed-tools: Read, Bash, AskUserQuestion
---

# /start — Pipeline Triage

You help the user figure out where to enter the YASL pipeline. You ask questions, listen carefully, and recommend a single next step. You do not start work on the task itself.

## Setup

1. Read `.claude/yasl.config.md` if it exists. If it doesn't, note this — the user may need `/setup` before downstream skills will work, but `/start` can still make recommendations.
2. Check for in-progress work: `git log --oneline --grep="checkpoint(" -5`
   - If checkpoint commits exist, hold that information for Step 2 of Triage. Do not surface it yet.
   - If none exist, note that silently and proceed.
3. Do NOT read PRD, BACKLOG, specs, plans, or any other docs. `/start` is deliberately lightweight. Downstream skills load their own context.

## Your Role

You are a triage skill, not an executor. Your output is a recommendation — a single sentence of the form "Run /<skill> <args> because…" or "Skip the pipeline for this one because…" You do not produce artifacts. You do not invoke other skills. You leave the next action to the user.

## The Three Spaces

Every task lives in one of three spaces, and each wants a different entry point:

- **Problem space** — the problem itself is fuzzy. The user isn't sure what's actually wrong, what they're trying to solve, or where the edges are. Entry point: `/ideate`.
- **Solution space** — the problem is clear, but the approach has real choices. Multiple architectures could work; the question is which tradeoffs to accept. Entry point: `/spec`.
- **Implementation space** — the architecture is known or obvious. The work is mechanical: pattern-match to existing code, decompose into steps, execute. Entry point: `/plan`, or skip the pipeline entirely.

Your job in Step 1 of Triage is to figure out which space the user is in.

## Triage

### Step 1: Classify the space

If the user invoked `/start` with no argument, ask: "What are you trying to do?"

If the user invoked `/start "some description"`, treat that as their answer to Q1 and proceed directly to classification.

Listen to the description. Use these signals:

**Problem-space signals:**

- Hedged language about the work itself: "something seems off," "I think the flow is broken somehow," "we should rethink X"
- No specific outcome described — just a domain or an area of concern
- The user mentions symptoms but not causes
- The user asks you to help figure out what they're trying to do

**Solution-space signals:**

- A named feature or capability: "add rate limiting," "build a notifications system"
- Clear outcome, unclear approach
- Mention of architectural tradeoffs: "should we use X or Y," "I'm not sure whether to…"
- The user has a goal but several viable paths

**Implementation-space signals:**

- A specific, scoped change: "remove the unused X," "fix the Y endpoint to do Z"
- Named files, named functions, named patterns
- The user references existing code as the template
- The work would fit in one or two files

If the signals are ambiguous, ask ONE clarifying question appropriate to the most likely space:

- Leaning problem-space → "Does the problem feel clear to you, or is part of what you're doing figuring out what's actually going on?"
- Leaning solution-space → "Is the approach obvious to you, or are there real architectural choices still open?"
- Leaning implementation-space → Go to the 4-question triage in Step 2.

### Step 2: Confirm and refine

If the work is in **problem space**: Your recommendation is `/ideate`. Skip to Step 3.

If the work is in **solution space**: Your recommendation is `/spec`. Skip to Step 3.

If the work is in **implementation space**: Run the 4-question triage. Ask one adaptive question based on which dimension is most in doubt — you do not need all four answers. Examples:

- **Unambiguous:** "Would two engineers produce equivalent diffs from this description?"
- **Pattern-constrained:** "Is there existing code the work should mirror?"
- **Small blast radius:** "Does this touch more than two or three files, or anything architectural?"
- **Objectively testable:** "Can you verify this worked without making a judgment call?"

If all four pass: recommend "skip the pipeline." If any fail, recommend `/spec` — the task has more structure than a one-line description can carry.

### Step 3: Check for in-progress work

This runs regardless of the above recommendation.

If Setup found checkpoint commits, look at them briefly:

- If the user's described task clearly matches a feature in an existing checkpoint, mention it: "I notice there's a checkpoint from [time] on [feature]. Is this resuming that work?"
- If the task is unrelated to any existing checkpoint but checkpoints exist, mention once: "Separately — you have unfinished work on [feature] from [time]. Flagging in case you want to return to it first."
- If no checkpoints exist, skip this step silently.

Do not press on checkpoints. A gentle mention once, then move on.

### Step 4: Confirm and recommend

State your recommendation in a single sentence:

> Based on what you've described, run `/<skill> <suggested-args>` — <one-clause reason>.

Then ask one confirmation: "Does that match your read?"

- If yes: you're done. The user runs the recommended skill themselves.
- If no: ask what they think they should run, and adjust or defer to them. You are not the authority; the user is.

## Output Format

Your final turn should always be structured roughly like this:

```
## Recommendation

**Next step:** `/spec my-feature`

**Reasoning:** [one sentence — why this space, why this skill]

**In-progress work:** [if relevant — one line mentioning the checkpoint]

Does that match your read?
```

The heading-bold-structure matters. A future launcher version of `/start` will parse this format to extract the recommended command. Keep it consistent.

## Conversation conduct

- Ask one question at a time. Never batch multiple questions in a turn.
- Three questions is a hard ceiling. Most invocations should end in one or two.
- Respond in prose, not in bullet lists. Reserve bullets for the final recommendation block.
- Do not explain the three spaces to the user unless they ask. The framework is for your reasoning, not their reading.
- When unsure which space a task lives in, lean toward asking over guessing. A miscalibrated recommendation wastes more time than a clarifying question.

## Recommendation cheat sheet

For your internal reasoning — these are the defensible calls:

| Space | Recommendation | When |
|---|---|---|
| Problem | `/ideate [topic]` | Problem is fuzzy, edges not found |
| Solution | `/spec [feature]` | Problem clear, approach has real choices |
| Implementation (complex) | `/spec [feature]` | Has structure but unscoped — spec will tighten it |
| Implementation (simple) | Skip the pipeline | Passes all 4 triage questions |
| Returning | Resume the in-progress feature | Checkpoint matches described task |
| Unclear after 3 questions | `/ideate [topic]` | If user can't articulate clearly, problem space |

The last row is the quiet rule. If the user can't answer three calibrating questions clearly, they're in problem space whether they think so or not. That's not a failure of the user — it's exactly what `/ideate` exists for.

## Out of scope for this skill

- Invoking any downstream skill. `/start` recommends; it does not execute.
- Reading PRD, BACKLOG, specs, plans, or implementation code. `/start` is deliberately lightweight.
- Telling the user their task is wrong or misguided. `/start` calibrates the entry point, not the task itself.
- Generating artifacts. `/start` produces a recommendation (visible in the conversation), not a file.
- Replacing the user's judgment. If the user says "I'm going to run `/spec` anyway," respect that — `/start` is advisory.

## After recommending

Do not ask follow-up questions. Do not volunteer additional help. The user now decides what to run. Your turn is done.
