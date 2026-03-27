---
name: brainstorm
description: "Use this before any creative work - creating features, building components, adding functionality, or modifying behavior. Explores user intent, requirements and design before implementation."
---

# Brainstorming Ideas Into Designs

## Preconditions
Make sure you work in @plan mode. Show warning to user if you not. 

## Overview

Help turn ideas into fully formed designs and specs through natural collaborative dialogue.

Start by understanding the current project context, then ask questions one at a time to refine the idea. Once you understand what you're building, present the design in small sections (200-300 words), checking after each section whether it looks right so far.

## The Process

**Understanding the idea:**
- Check out the current project state first (files, docs, recent commits)
- Ask questions one at a time to refine the idea
- Prefer multiple choice questions when possible, but open-ended is fine too
- Only one question per message - if a topic needs more exploration, break it into multiple questions
- Focus on understanding: purpose, constraints, success criteria

**Exploring approaches:**
- Propose 2-3 different approaches with trade-offs
- Present options conversationally with your recommendation and reasoning
- Lead with your recommended option and explain why

**Presenting the design:**
- Once you believe you understand what you're building, present the design
- Break it into sections of 200-300 words
- Ask after each section whether it looks right so far
- Cover: architecture, components, data flow, error handling, testing
- Be ready to go back and clarify if something doesn't make sense

## After the Design
After design is validated, use questions/AskUserQuestion tool:

```json
{
  "questions": [{
    "question": "Design looks complete. What's next?",
    "header": "Next step",
    "options": [
      {"label": "Write plan", "description": "Create docs/plans/YYYY-MM-DD-<topic>-plan.md with implementation steps via /planning:make"},
      {"label": "Plan mode", "description": "Enter plan mode for structured implementation planning"},
      {"label": "Start now", "description": "Begin implementing directly"}
    ],
    "multiSelect": false
  }]
}
```

- **Write plan**: invoke `/plan` command to create the plan file. Pass brainstorm context (discovered files, selected approach, design decisions) as arguments so the plan command has full context without re-asking questions
- **Plan mode**: uses @plan tool for detailed planning with user approval workflow
- **Start now**: proceeds directly if design is simple enough using @build

**Documentation:**
- Use @build and write the validated design to `docs/plans/YYYY-MM-DD-<topic>-design.md`
- Use elements-of-style:writing-clearly-and-concisely skill if available
- Commit the design document to git

**Implementation (if continuing):**
- Ask: "Ready to set up for implementation?"
- Use writing-plans skill to create detailed implementation plan

## Key Principles

- **One question at a time** - Don't overwhelm with multiple questions
- **Multiple choice preferred** - Easier to answer than open-ended when possible
- **YAGNI ruthlessly** - Remove unnecessary features from all designs
- **Explore alternatives** - Always propose 2-3 approaches before settling
- **Incremental validation** - Present design in sections, validate each
- **Be flexible** - Go back and clarify when something doesn't make sense
