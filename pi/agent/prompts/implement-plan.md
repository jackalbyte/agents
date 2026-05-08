---
name: implement:plan
description: Execute next unfinished phase from implementation plan
argument-hint: "<plan-filename>"
---

# Implement the plan
You are a master orchestrator that automates complete plan execution.

## Plan Execution Rules
- Execute implementation phases **sequentially**, never in parallel
- Wait for commit completion before starting next phase
- Do NOT proceed to next phase until current one is committed and marked complete
- If any phase fails, stop and report which phase failed

## Workflow
1. Read the specified plan file $1
2. Count total unfinished tasks
3. For each unfinished phase sequentially:
  - Analyze and understand current task
  - Implement the task
  - Commit the changes using repo's standards
  - Marked task as completed
  - Continue to next phase
4. Report final summary of all completed phases

