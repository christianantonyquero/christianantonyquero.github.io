---
title: "One assistant, five agents: structuring a multi-agent coding pipeline for Django"
date: 2026-11-04 10:00:00 +0400
description: "Why I split a single Claude Code session into orchestrator, coder, reviewer, lint and test agents, and how plain markdown guideline files keep each one in its lane."
tags: [ai, claude-code, agents, django]
series: AI-assisted engineering
---

A single AI coding session that plans, writes, reviews and tests its own work runs into a familiar problem: **it grades its own homework.** The same context that produced a bug is the one asked to find it. On a large Django codebase, that context also fills up quickly with files, test output and lint noise until the model loses track of the original task.

My fix was to stop treating the assistant as one generalist and split it into roles, the way a real team works.

## The five roles

| Agent | Owns | Does not touch |
|---|---|---|
| **Orchestrator** | Breaks the task into steps, delegates, decides when it's done | Writing code |
| **Coder** | Implements one well-scoped change | Reviewing or approving its own work |
| **Reviewer** | Reads the diff with fresh context, flags correctness and design issues | Rewriting the change itself |
| **Lint** | Style, typing, import order, project conventions | Behaviour |
| **Test** | Writes and runs tests, reports failures with evidence | Changing production code to make tests pass |

The "does not touch" column matters more than the "owns" column. Most of the value comes from the boundaries. A reviewer that can't edit has to explain the problem clearly. A test agent that can't change production code can't quietly weaken an assertion to get to green.

## Guideline files

Each role gets a short markdown file instead of one giant prompt:

```text
AGENTS.md              # roles, hand-offs, definition of done
guidelines/
  lint.md              # conventions the lint agent enforces
  test.md              # what a good test looks like here
LESSONS.md             # shared, append-only learnings
```

`AGENTS.md` is the contract. A simplified version of its shape:

```markdown
## Orchestrator
- Input: a ticket or spec. Output: an ordered list of steps, each small
  enough for one coder pass.
- Hand each step to the coder, then the reviewer, then lint and test.
- Done = reviewer has no blocking comments AND tests pass locally.

## Reviewer
- Start from the diff, not from the coder's explanation.
- Every comment is blocking or non-blocking. Never edit files.
```

Keeping each file small and role-specific has two benefits. Each agent loads only the rules it needs, which keeps context lean. And when an agent misbehaves, I know exactly which file to fix.

## LESSONS.md: memory that survives the session

Agents don't remember yesterday, so the same mistakes kept coming back in new sessions: project conventions they had already been corrected on once. So every agent can **append** to a shared `LESSONS.md`, and every agent **reads** it at the start.

The rule is that a lesson must be specific and checkable. "Be careful with migrations" is useless. "Data migrations go in a separate migration from schema changes" is something the reviewer can enforce.

## Handling ambiguous specs

Before the pipeline, I set up a `CLAUDE.md` framework for implementing features systematically. The most useful part was a **risk-based rule for ambiguity**:

- **Low risk** (naming, internal structure): make a reasonable assumption, write it down, keep going.
- **High risk** (data model, external API contracts, anything hard to roll back): stop and ask.

Without this rule an assistant either asks about everything, which is exhausting, or assumes everything, which is dangerous. The other non-negotiable: **the local test suite must pass before any Playwright end-to-end tests run.** E2E runs are slow and expensive, so they should never be where you find a unit-level bug.

## What I'd tell someone starting out

1. **Start with boundaries, not prompts.** Decide what each agent is forbidden to do first.
2. **Keep guideline files short and versioned.** Treat them like code: they get reviewed and they change when reality changes.
3. **Make memory explicit.** If you want the system to learn, give it a file it has to read and write.
4. **Watch the context budget.** Splitting roles is partly a context-management strategy. More on that in the next post, where the pipeline slowed to a crawl.
