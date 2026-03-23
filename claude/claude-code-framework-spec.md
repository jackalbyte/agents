# Claude Code Development Framework — Specification

## Overview

A structured, multi-stage framework for software development using Claude Code. The framework enforces a strict workflow: **Brainstorm → Plan → Implement**, with each stage producing a well-defined artifact that feeds the next. Context is kept lean throughout by using isolated subagents and dynamically loaded skills.

---

## Core Principles

- **Staged handoffs** — each stage has a defined input and output; no stage starts without the previous one's artifact
- **Lean context** — subagents receive only what they need; design knowledge and test logic never pollute implementation context
- **Fail loud** — missing prerequisites cause an immediate, clear stop with instructions, not silent failures
- **Human-gated** — brainstorm and planning are always human-in-the-loop; implementation can be invoked manually per task

---

## Stages

### 1. Brainstorm

**Purpose:** Explore the problem space, discuss solutions collaboratively, and converge on a clear approach.

**Trigger:** Human invokes `/brainstorm` command.

**Input:** A raw idea as an inline argument or a path to a file.
```
/brainstorm "add OAuth login to the app"
/brainstorm docs/ideas/oauth-notes.md
```

**Process:**
- Claude asks clarifying questions and discusses alternative solutions
- Dialogue continues until both parties converge on an approach
- Claude writes the brainstorm artifact to `docs/brainstorms/YYYYMMDD-topic.md`

**Output — Brainstorm Artifact Schema:**
```markdown
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

**Rules:**
- The command does not conclude until the artifact file is written
- The artifact schema is fixed; Claude must not omit sections

---

### 2. Planning

**Purpose:** Turn the brainstorm artifact into a concrete, step-by-step implementation plan.

**Trigger:** Human invokes `/plan` command with the brainstorm artifact path.
```
/plan docs/brainstorms/20240315-oauth.md
```

**Process:**
- Claude reads the brainstorm artifact
- Invokes relevant design skills based on what the artifact requires (see Design Skills below)
- Produces a step-by-step plan with checkboxes
- Human reviews and approves before anything proceeds
- Plan is written to `docs/plans/YYYYMMDD-topic-plan.md`

**Output — Plan File Schema:**
```markdown
## Goal
One-sentence summary of what this plan delivers.

## Context
Reference to source brainstorm artifact.

## Tasks
- [ ] Task 1 description
- [ ] Task 2 description
- [ ] Task 3 description

## Notes
Any decisions, caveats, or dependencies relevant to implementation.
```

**Rules:**
- Tasks use `- [ ]` / `- [x]` checkbox syntax — this is the state contract with the orchestrator
- Human must explicitly approve the plan before `/implement` is invoked
- Design skills are invoked dynamically; not all plans require all skills

---

### 3. Implementation

**Purpose:** Execute the plan task by task using isolated subagents.

**Trigger:** Human invokes `/implement` command with the plan file path.
```
/implement docs/plans/20240315-oauth-plan.md
```

**Process:**
1. Orchestrator checks for `docs/agent-runtime.md` — if missing, runs bootstrap logic inline and writes the file before proceeding
2. Orchestrator reads the plan, finds the first `- [ ]` task
3. Spawns an Implement Subagent with only that task and relevant file context
4. Implement Subagent completes the task and reports back
5. Orchestrator spawns a Tester Subagent to verify the change
6. If tests pass: orchestrator marks the task `- [x]` in the plan file
7. If tests fail: orchestrator re-invokes Implement Subagent with the failure output
8. Human re-invokes `/implement` for the next task (manual loop — one task per invocation)

**Rules:**
- Orchestrator is the single gate for all precondition checks
- Subagents trust that `docs/agent-runtime.md` exists; they do not re-check
- Implementation stops and surfaces errors clearly; it never silently skips tasks

---

## Entities

### Commands

| Command | Input | Output |
|---|---|---|
| `/brainstorm` | Idea string or file path | `docs/brainstorms/YYYYMMDD-topic.md` |
| `/plan` | Brainstorm artifact path | `docs/plans/YYYYMMDD-topic-plan.md` |
| `/implement` | Plan file path | Updated plan file with tasks checked off |
| `/bootstrap` | None (auto-invoked by `/implement` if needed) | `docs/agent-runtime.md` |

---

### Skills (dynamically invoked during Planning)

Skills are loaded by the Planning command only when the brainstorm artifact requires them. They are never loaded all at once.

#### `design-db-schema`
- **Invoked when:** the artifact involves persistent data storage
- **Carries:** naming conventions, migration strategy, indexing rules, ORM preferences
- **Produces:** DB schema section appended to the plan

#### `design-api`
- **Invoked when:** the artifact involves exposing or consuming an HTTP API
- **Carries:** REST/GraphQL conventions, auth patterns, versioning strategy, error response format
- **Produces:** API design section appended to the plan

#### `design-queue`
- **Invoked when:** the artifact involves async processing or event-driven communication
- **Carries:** broker conventions, retry/DLQ patterns, idempotency requirements, message schema rules
- **Produces:** Queue design section appended to the plan

---

### Subagents (spawned during Implementation)

#### Implement Subagent
- **Spawned by:** Orchestrator (inside `/implement`)
- **Receives:** single task description, relevant source files, `docs/agent-runtime.md`
- **Produces:** code changes for that task only
- **Knows nothing about:** the rest of the plan, other tasks, test infrastructure

#### Tester Subagent
- **Spawned by:** Orchestrator after each Implement Subagent completes
- **Receives:** task description, changed files, `docs/agent-runtime.md`
- **Process:**
  1. Reads `docs/agent-runtime.md` for test command
  2. Runs tests scoped to changed files/modules
  3. Returns structured pass/fail report with output
- **Knows nothing about:** implementation details, the full plan, other tasks

---

## Shared Runtime File

### `docs/agent-runtime.md`

Written once by the bootstrap logic (auto-triggered by `/implement` if missing). Consumed by all subagents. Never re-generated unless explicitly re-run.

**Schema:**
```markdown
## Runtime

test_command: ./gradlew test
run_command: ./gradlew bootRun
stop_command:
test_scope_flag: --tests {classname}
package_manager: gradle
notes: Spring Boot 3.x project
```

**Bootstrap discovery order:**
1. Read `CLAUDE.md`
2. Read `AGENT.md`
3. Read `README.md`
4. Infer from project structure (detect `gradlew`, `package.json`, `Makefile`, `pyproject.toml`, etc.)

**Rules:**
- If discovery finds no usable information, bootstrap writes the file with empty values and warns the user to fill it in manually
- Subagents read this file but never write to it

---

## File Structure

```
docs/
  brainstorms/
    YYYYMMDD-topic.md         ← brainstorm artifact
  plans/
    YYYYMMDD-topic-plan.md    ← implementation plan
  agent-runtime.md            ← shared runtime config (bootstrap output)
```

---

## Failure Handling

| Failure | Behaviour |
|---|---|
| `docs/agent-runtime.md` missing | `/implement` runs bootstrap inline, writes file, then continues |
| Bootstrap cannot determine test command | Writes file with empty fields, warns user to fill in manually |
| Implement Subagent fails | Orchestrator surfaces error, stops, waits for human intervention |
| Tester Subagent reports failure | Orchestrator re-invokes Implement Subagent with failure output attached |
| Tester fails twice on same task | Orchestrator stops and surfaces to human — no infinite retry |

---

## Design Decisions & Rationale

| Decision | Rationale |
|---|---|
| Commands for Brainstorm and Plan | Both are human-in-the-loop and human-initiated; commands are the right fit for explicit workflow gates |
| Skills for design specialists | Design needs vary per artifact; skills are loaded dynamically rather than polluting all plans with all conventions |
| Orchestrator + subagents for implementation | Failure isolation — a failing subagent doesn't corrupt loop state; each subagent gets a clean context window |
| Manual loop (one task per `/implement` call) | Keeps human in control; easy to debug; automated looping can be added later once the single-task flow is proven |
| Flat file for runtime config | Simpler and more debuggable than a persistent subagent; all consumers read the same source of truth; fixing a wrong value means editing one file |
| Bootstrap auto-triggered by `/implement` | Eliminates user responsibility for ordering; no separate step to forget |
