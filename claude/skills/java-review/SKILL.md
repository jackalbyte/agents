---
name: java-review
description: Deep Java code review skill for the code-reviewer agent. Provides focused, Java/Spring-specific guidance across five review dimensions. Invoke with a focus area or leave open for a full review. Reads supporting files on demand to avoid loading all guidance at once.
allowed-tools: Read, Glob, Grep, LSP
---

# Java Review

You are performing a deep Java-specific code review to complement standard review criteria. This skill provides detailed checklists and patterns for common Java/Spring pitfalls that survive static analysis and basic code review.

## How to Use This Skill

The skill is organized into five supporting files. Read only the ones relevant to what you're reviewing:

| Focus area | File to read | When to invoke |
|---|---|---|
| Performance | `performance.md` | Hot paths, processing loops, high-throughput services, batch jobs |
| Architecture | `architecture.md` | New classes/services, layer changes, hexagonal boundary crossings |
| Correctness | `correctness.md` | Logic-heavy code, null handling, Optional usage, JPA entities |
| Security | `security.md` | Endpoints, query construction, auth, data exposure |
| Testing | `testing.md` | New or modified test classes |

**Default behavior:** If no focus is specified, determine which files are relevant by scanning the changed code briefly, then read and apply all relevant guides.

## Navigation

- Performance anti-patterns → Read `performance.md`
- Spring/hexagonal architecture violations → Read `architecture.md`
- Java correctness pitfalls → Read `correctness.md`
- Security vulnerabilities → Read `security.md`
- Test quality → Read `testing.md`

## Output

Append findings to the main review under the relevant category section. Use the standard severity markers:
- `❌ Critical` — must fix before merge
- `⚠️ Warning` — strongly recommended
- `💡 Suggestion` — worth considering

Always include: class/method reference, why it matters, corrected snippet when non-trivial.
