---
description: Create an execution plan. Run this when user need a plan or ask to create it, when user needs todo list. 
---

# Write a plan
Write a plan that a worker agent can execute without access to the planning conversation. 
Every todo must be self-contained — a worker reading only the todo and the plan artifact must produce the correct result.

## Structure
Structure should contains tasks list. Example below shows the structure example:

```markdown
# Plan: [plan name]
## Tasks list
- [ ] Task 1 - [Task goal]
...
- [ ] Task n - [Task goal]

### Task 1 — [Task goal] 
...
### Task n — [Task goal] 
```

## Each task in the plsn is self-contained
A worker reads: (1) the todo body, (2) the plan artifact, (3) existing code. That's it. They don't read other todos. So:
- Reference the plan path in every todo
- List all files to create or modify
- Note which existing files to read for context
- Include any conventions discovered during planning

## Tests maintaining 
Every task in the plan must be **independently implementable with a fully green test suite before and after**. This means:

- If a task changes a public interface (constructor signature, method signature, interface contract, function parameters), it must also update **every existing call site** in the same task — including production code, test helpers, fixtures, and factories — using a safe default or placeholder where the final value comes from a later task.
- Before writing a task that touches a signature, search the codebase for all usages (`grep -rn 'new ClassName\|->method\|::method'`) and list them exhaustively in the task's scope.
- Never defer "fix broken call sites" to a later task. A later task may refine the value, but the codebase must compile and all tests must pass immediately after each task is applied.

Apply this rule to any change that can break existing code: renamed methods, changed return types, new required parameters, moved files, schema changes reflected in ORM entities, etc.

## Result
Save the plan to `docs/tasks/<task-name>/PLAN.md` folder.
