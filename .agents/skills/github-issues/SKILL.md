---
name: github-issues
description: Fetch and rank GitHub issues from repositories owned only by the authenticated account, with modes for Recent, Quick, Complex, and Boring tasks plus optional programming-language filtering.
---

# GitHub Issues

Use this skill when the user asks to collect GitHub issues from repositories they own, especially via a slash command like `/github-issues Recent|Quick|Complex|Boring`.

## Core rules

1. Always answer in the same language as the user's request.
2. Only use repositories that are owned by the authenticated user account.
   - Do not include organization repositories.
   - Do not include repositories owned by other users.
3. Prefer open issues as actionable tasks.
4. Sort selected issues by creation date (`created_at`). Use newest-first ordering unless the user explicitly asks otherwise.
5. If the user provides preferred programming languages, only include repositories whose code matches those languages.
   - Exclude repositories in other languages.
   - If multiple languages are provided, accept any matching repository.
6. If the issue type is missing, treat it as `Recent`.
7. If the issue type cannot be recognized, say that the type was not recognized and fall back to `Recent`.

## Input parsing

Interpret the first recognized mode token anywhere in the request as one of:

- `Recent`
- `Quick`
- `Complex`
- `Boring`

If no valid mode is found, use `Recent` and mention the fallback.

Treat any additional language names in the request as a repository language filter.

## Mode definitions

### Recent
Return all matching issues from the user's own repositories.

### Quick
Return issues that are likely doable in about one hour or less.
Typical signals:
- small bug fixes
- narrow UI or API adjustments
- low-risk, localized changes
- well-scoped improvements with limited code impact

Exclude large refactors, broad design work, and tasks that clearly need more than an hour.

### Complex
Return the most interesting non-template engineering work.
Typical signals:
- new features that require design decisions
- non-boilerplate implementation work
- architecture or integration tasks
- problems that need careful planning

Prefer issues that are not simple connector/add-on/template-style work.

### Boring
Return refactoring and cleanup work.
Typical signals:
- codebase cleanup
- repetitive refactors
- mechanical improvements
- technical-debt reduction
- long, tedious maintenance tasks

## Selection workflow

1. Read the request.
2. Detect the mode.
   - If missing, use `Recent`.
   - If unrecognized, mention the fallback to `Recent`.
3. Detect any requested programming languages.
4. Get the authenticated GitHub account identity.
5. List repositories owned by that account only.
6. If language filters are present, keep only repositories that match them.
7. Collect open issues from the remaining repositories.
8. Filter issues according to the selected mode.
9. Sort the final issues by creation date.
10. Return the result in the exact markdown format below.

## Issue format

For every issue, output:

```markdown
## Issue title
Short summary of the task.
GitHub issue link
```

Keep one issue block per issue, separated by a blank line.
Do not add extra commentary unless needed to explain an unrecognized mode fallback or an empty result.

## Empty-result handling

If nothing matches:
- say that no issues were found for the selected mode and filters
- keep the answer in the same language as the user

## Language guidance

When the user specifies preferred programming languages:
- only propose repositories in those languages
- do not mention repositories outside the requested language set
- if nothing matches, say so clearly
