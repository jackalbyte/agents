---
description: Execute the next unchecked task in a plan file using isolated subagents
argument-hint: "[path to plan file, e.g. docs/plans/20240315-oauth-plan.md]"
---

# Implement

You are the orchestrator for task-by-task implementation. You execute **one task per invocation** and gate everything through precondition checks. Never silently skip steps or swallow errors.

## Input

Plan file path: $ARGUMENTS

**Fail loud:** If no argument is provided or the file does not exist, stop immediately:
> `/implement` requires a plan file path. Run `/plan` first to produce one.

## Step 1 — Bootstrap gate

Check whether `docs/agent-runtime.md` exists in the current working directory.

**If it does not exist:** Run bootstrap logic inline before proceeding:
1. Attempt to read `CLAUDE.md`, then `AGENT.md`, then `README.md` (in that order, stop at the first one that exists and contains useful runtime info)
2. Infer from project structure: detect `gradlew` → Gradle, `package.json` → npm/yarn/pnpm, `Makefile` → make, `pyproject.toml` → Python, `pom.xml` → Maven, `Cargo.toml` → Cargo, `go.mod` → Go
3. Write `docs/agent-runtime.md` using this exact schema:

```
## Runtime

test_command: [e.g. ./gradlew test]
run_command: [e.g. ./gradlew bootRun]
stop_command: [e.g. ]
test_scope_flag: [e.g. --tests {classname}]
package_manager: [e.g. gradle]
notes: [e.g. Spring Boot 3.x project]
```

4. If you cannot determine values, write the file with empty values and warn the user:
> ⚠️ `docs/agent-runtime.md` was created but some fields are empty. Please fill them in manually before re-running `/implement`.
> Then stop.

**If it exists:** proceed.

## Step 2 — Find the next task

Read the plan file. Find the **first** line matching `- [ ]`.

If no unchecked tasks remain, stop and tell the user:
> All tasks in this plan are complete. Nothing left to implement.

Extract the task description. Show it to the user:
> **Next task:** [task description]
> Spawning implement subagent...

## Step 3 — Spawn Implement Subagent

Spawn the `implement-subagent` agent. Provide it with:
- The task description
- The content of `docs/agent-runtime.md`
- Paths to source files relevant to this task (use Glob/Grep to identify them based on the task description)

The subagent will produce code changes and return a summary.

## Step 4 — Spawn Tester Subagent

Spawn the `tester-subagent` agent. Provide it with:
- The task description
- The list of files changed by the implement subagent
- The content of `docs/agent-runtime.md`

The subagent will run tests scoped to the changed files and return a structured pass/fail report.

## Step 5 — Handle test result

**If tests pass:**
- Mark the task as complete in the plan file: change `- [ ]` to `- [x]` for that specific task line
- Report to the user:
> ✅ Task complete: [task description]
> Plan updated. Re-run `/implement [plan path]` to execute the next task.

**If tests fail (first failure):**
- Re-spawn the `implement-subagent` with the original task description **plus** the full test failure output
- Then re-spawn `tester-subagent` again

**If tests fail a second time:**
- Stop. Do not retry again. Report to the user:
> ❌ Task failed after two attempts: [task description]
> Test output: [failure details]
> Human intervention required. Fix the issue and re-run `/implement [plan path]`.

## Rules

- One task per invocation. Never execute more than one `- [ ]` task per call.
- Never mark a task `- [x]` unless the tester subagent confirmed pass.
- Never silently swallow errors or proceed past a blocker.
- The plan file is the single source of truth for task state.
