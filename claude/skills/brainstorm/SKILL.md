---
name: brainstorm
description: "Use this before any creative work - creating features, building components, adding functionality, or modifying behavior. Explores user intent, requirements and design before implementation."
---

# Brainstorming Ideas Into Designs

## Overview

Help turn ideas into fully formed designs and specs through natural collaborative dialogue.

Start by understanding the current project context, then ask questions one at a time to refine the idea. Once you understand what you're building, present the design in small sections (200-300 words), checking after each section whether it looks right so far.

## The Process

**Understanding the idea:**
- Check out the current project state first (files, docs, recent commits)
- Ask questions one at a time to refine the idea using the AskUserQuestion tool
- Prefer multiple choice options in AskUserQuestion when possible; use open-ended ("Other") as the fallback
- Only one question per AskUserQuestion call - if a topic needs more exploration, ask follow-up questions in subsequent turns
- Focus on understanding: purpose, constraints, success criteria

**Exploring approaches:**
- Propose 2-3 different approaches with trade-offs using AskUserQuestion with option previews when helpful
- Lead with your recommended option and explain why

**Presenting the design:**
- Once you believe you understand what you're building, present the design
- Break it into sections of 200-300 words
- After each section, use AskUserQuestion to ask whether it looks right so far (Yes / Needs changes)
- Cover: architecture, components, data flow, error handling, testing
- Be ready to go back and clarify if something doesn't make sense

## After the Design

**Create an artifact:**
- Write the validated specification to `docs/tasks/<task-name>/SPEC.md`
- Commit the design document to git

## Key Principles

- **Use AskUserQuestion tool** - Always ask questions via the tool, never as plain text
- **One question at a time** - One AskUserQuestion call per turn
- **Multiple choice preferred** - Easier to answer than open-ended when possible
- **YAGNI ruthlessly** - Remove unnecessary features from all designs
- **Explore alternatives** - Always propose 2-3 approaches before settling
- **Incremental validation** - Present design in sections, validate each
- **Be flexible** - Go back and clarify when something doesn't make sense
