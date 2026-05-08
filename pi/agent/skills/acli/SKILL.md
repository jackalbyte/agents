---
description: Work with Jira issues using Atlassian CLI (acli). Retrieve, update, and manage work items.
---

# Atlassian CLI (acli) Skill

Use this skill when working with Jira issues and need to fetch or update issue details.

## Common Commands

### View Issue Details

Get full details of a Jira issue:

```bash
acli jira workitem view CRAW-1330
```

Returns: Key, Type, Summary, Status, Assignee, Description, and other fields.

### Edit Issue Description

Update an issue's description:

```bash
acli jira workitem edit --key "CRAW-1330" --description "New description text" --yes
```

Use `--yes` flag to skip confirmation.

### Edit Issue Summary

Update an issue's summary/title:

```bash
acli jira workitem edit --key "CRAW-1330" --summary "New Summary" --yes
```

### Assign Issue

Assign issue to a user:

```bash
acli jira workitem assign --key "CRAW-1330" --assignee "user@example.com"
```

Use `@me` to self-assign.

### Search Issues

Find issues using JQL:

```bash
acli jira workitem search --jql "project = CRAW AND status = 'In Progress'"
```

### Transition Issue

Move issue to a different status:

```bash
acli jira workitem transition --key "CRAW-1330" --transition "In Progress"
```

### Add Label

Add labels to an issue:

```bash
acli jira workitem edit --key "CRAW-1330" --labels "bug,urgent" --yes
```

### Update via JQL

Update multiple issues using JQL query:

```bash
acli jira workitem edit --jql "project = TEAM" --assignee "user@example.com" --yes
```

## Patterns

### Get and Update

When asked to update an issue based on changes made:

1. Use `acli jira workitem view ISSUE-KEY` to get current details
2. Review what changed (check git, commit messages, etc.)
3. Use `acli jira workitem edit --key ISSUE-KEY --description "..." --yes` to update

### Batch Operations

For bulk updates, use `--jql` instead of `--key`:

```bash
acli jira workitem edit --jql "project = CRAW AND status = Draft" --summary "Updated" --yes
```

### Description Formatting

- Keep descriptions business-focused when updating from technical changes
- Use clear language for product managers, not implementation details
- For multi-line descriptions, use proper newline formatting
- Always include background/context sections when relevant

## Tips

- Use `--yes` flag to skip confirmation prompts
- Check `acli jira workitem --help` for all available options
- All workitem commands support `--json` for machine-readable output
- Use `--from-json` to batch edit via JSON file for complex updates
