---
description: Explore a problem space collaboratively and produce a brainstorm artifact
argument-hint: "[idea string or path to idea file]"
---

# Brainstorm

You are helping the user explore a problem and converge on a clear approach. This is a human-in-the-loop collaborative session — do not skip to solutions.

## Input

$ARGUMENTS

If the argument is a file path (ends with `.md` or exists as a file), read it first. Otherwise treat the argument as a raw idea string. If no argument is provided, ask the user what they want to explore.

## Process

**Step 1 — Understand the problem**
- Summarize your understanding of the problem in 2–3 sentences
- Identify any ambiguities, unstated assumptions, or missing context

**Step 2 — Ask clarifying questions**
Present a numbered list of clarifying questions. Focus on:
- What problem is actually being solved (vs. the stated solution)?
- Who are the users/consumers of the thing being built?
- What are the key constraints (tech stack, time, team, compliance)?
- What does success look like?
- What has already been tried or ruled out?

Wait for the user's answers before proceeding.

**Step 3 — Explore alternatives**
Once you have enough context, present 2–4 alternative approaches. For each:
- Describe the approach concisely
- List its key trade-offs (pros and cons)
- Note when it would be the right choice

**Step 4 — Converge**
Facilitate convergence. Ask the user which approach they prefer and why. If they're uncertain, give your recommendation with clear reasoning. Continue the dialogue until both of you agree on a specific approach.

**Step 5 — Write the artifact**
Once you have converged, determine today's date and derive a short topic slug from the idea (e.g., `oauth-login`, `health-check-endpoint`).

Write the artifact to `docs/brainstorms/YYYYMMDD-{topic}.md` using **exactly** this schema — do not omit any section:

```
## Problem
Clear description of the problem being solved.

## Constraints
Technical, business, or time constraints that apply.

## Chosen Approach
The agreed solution and why it was selected.

## Rejected Alternatives
Other approaches considered and why they were ruled out.

## Open Questions
Anything unresolved that planning needs to address.
```

**This command does not conclude until the artifact file is written.** Confirm the file path to the user when done.
