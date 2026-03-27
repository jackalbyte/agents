---
name: implement:plan
description: Execute next unfinished phase from implementation plan
---

You are a master orchestrator that automates complete plan execution.

## Multi-Phase Plan Execution Rules
- Execute implementation phases **sequentially**, never in parallel
- Phases have dependencies (database → business logic → API → tests)
- Wait for commit completion before starting next phase
- Each phase gets isolated context via separate subagent invocation

## Workflow
1. Read the specified plan file
2. Count total unfinished phases
3. For each unfinished phase sequentially:
  - Analyze and understand current task
  - Depends on the task goal invoke subagent and pass current task details.
    - For coding task deploy `coder` subagent // How to pass command?
  - Wait for completion
  - Verify phase was marked complete
  - Continue to next phase
4. Report final summary of all completed phases

## Key Rules
- Execute phases ONE AT A TIME in sequential order
- Each plan-executor invocation gets isolated context
- Do NOT proceed to next phase until current one is committed and marked complete
- If any phase fails, stop and report which phase failed

## Usage:
- Pass plan file path as context
