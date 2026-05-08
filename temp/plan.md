# Prompt: Generate a Structured Implementation Plan from a Design Document

You are an expert software engineer. Given a design document, generate a structured implementation plan following the rules below.

---

## Rules

### Structure

- Each section represents one logical unit of change (one file or one closely related group of files).
- Order sections by dependency: things that others depend on come first.
- Number sections sequentially starting from 1.
- Add a **Checklist** after the Overview with one checkbox per section, linking to it by number.
- End with a **Summary table** listing every changed file and a one-line description of the change.

### Each section must contain

1. **File path(s)** affected.
2. **What changes** — describe the code change concisely. Include code snippets where the change is non-obvious.
3. **Tests subsection** (if tests exist for this file) — placed at the bottom of the section, marked `> Update before implementation.`
   - Explain **why** existing tests break.
   - List specific test updates as checkboxes.
   - List new test cases as checkboxes with a one-line description of what each case covers.
   - Do **not** group tests in a separate top-level section — keep them co-located with the code they test.
   - If a test file is affected by multiple sections, place it in the section that owns the most significant change, and add a brief note explaining the cross-section dependency.

### Database migrations

- If a schema change is involved, include a dedicated migration section.
- Prefer preserving existing data over truncation.
- If backfill is needed: add column as nullable → UPDATE → enforce NOT NULL.
- Describe the exact SQL steps in order.

### External dependencies

- If a change requires updating an external library or contract owned by another team, call it out explicitly in its own section.
- Include the required change, the risk of deploying out of order, and a coordination note.

### Tone and format

- Use checkboxes (`- [ ]`) for every actionable item so the plan can be used as a task tracker.
- Keep code snippets minimal — only show the changed signature or block, not the whole file.
- Be explicit about renamed methods: state old name → new name and list every call site that must be updated.
- Mark sections that require no code changes explicitly (e.g. `> No code changes required here.`) so the reader does not need to re-verify.

---

## Input

Provide:
1. The design document (architecture decisions, data model, message contracts, output format).
2. Relevant existing source files (entities, handlers, services, repositories, DTOs).
3. Relevant existing test files.

---

## Output

A single Markdown document with:
- `## Overview` — one short paragraph summarising what the plan implements and why.
- `## Checklist` — one checkbox per section.
- `## 1. … ## N.` — one section per logical change unit, each with code changes and co-located test instructions.
- `## Summary of changed files` — table with columns `File` and `Change`.
