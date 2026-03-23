---
description: Turn a brainstorm artifact into a concrete implementation plan
argument-hint: "[path to brainstorm artifact, e.g. docs/brainstorms/20240315-oauth.md]"
---

# Plan

You are turning a brainstorm artifact into a concrete, step-by-step implementation plan. This is a human-in-the-loop stage — the human must approve before the plan is written.

## Input

Brainstorm artifact path: $ARGUMENTS

**Fail loud:** If no argument is provided, or the file does not exist, stop immediately and tell the user:
> `/plan` requires a brainstorm artifact path. Run `/brainstorm` first, then pass the output file path as the argument.

## Process

**Step 1 — Read the artifact**
Read the brainstorm artifact at the provided path. Summarize the problem, chosen approach, and key constraints so the user can confirm you've understood correctly.

**Step 2 — Invoke design skills (dynamically)**
Based on what the artifact requires, invoke the relevant design skills. Only invoke the skills that apply — do not invoke all of them blindly.

- If the artifact involves **persistent data storage** (databases, models, migrations): invoke the `design-db-schema` skill
- If the artifact involves **exposing or consuming an HTTP API** (REST, GraphQL, webhooks): invoke the `design-api` skill
- If the artifact involves **async processing or event-driven communication** (queues, workers, events): invoke the `design-queue` skill

For each applicable skill, work through its guidelines and produce the corresponding design section that will be appended to the plan.

**Step 3 — Draft the plan**
Produce a complete plan using `- [ ]` checkbox syntax for every task. Tasks must be:
- Independently executable by a subagent (no task should depend on uncommitted state from another)
- Scoped to a single logical change (a task is not "implement the feature")
- Ordered so earlier tasks do not block later ones where possible

**Step 4 — Present and get approval**
Show the full draft plan to the user. Ask explicitly:
> Does this plan look right? Approve it and I'll write it to disk, or tell me what to change.

Wait for explicit approval. Do not write the file until the user approves.

**Step 5 — Write the plan file**
Determine today's date and derive a short topic slug from the brainstorm filename or problem description.

Write the plan to `docs/plans/YYYYMMDD-{topic}-plan.md` using **exactly** this schema:

```
## Goal
One-sentence summary of what this plan delivers.

## Context
Reference to source brainstorm artifact: [path]

## Tasks
- [ ] Task 1 description
- [ ] Task 2 description
- [ ] Task 3 description

## Notes
Any decisions, caveats, or dependencies relevant to implementation.
```

Append any design skill output (DB schema, API design, queue design) as additional sections after `## Notes`.

Confirm the file path to the user when done.
