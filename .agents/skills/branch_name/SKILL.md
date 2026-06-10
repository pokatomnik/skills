---

name: branch-name-generator
version: 1.0.0
description: Generate exactly one concise Git branch name from a project purpose and a git diff.
language: en_US

inputs:

- name: Project purpose
  type: text
  required: true
  description: Short explanation of what the project does and what domain terms are meaningful.
- name: git diff output
  type: text
  required: true
  description: Full or partial git diff used to infer the intent of the change.

output:
format: plain_text
min_lines: 1
max_lines: 1
allowed_formats:
- feature/[NAME]
- bugfix/[NAME]
name_format:
placeholder: NAME
format: "[word1]-[word2]-[word3]-...-[wordN]"
words_min: 1
words_max: 4
casing: lowercase
separator: hyphen
charset: ascii
allowed_patterns:
- "^feature/[a-z0-9]+(-[a-z0-9]+){0,3}$"
- "^bugfix/[a-z0-9]+(-[a-z0-9]+){0,3}$"

restrictions:

- Return exactly one branch name.
- Do not return explanations, comments, markdown, quotes, backticks, bullets, or extra text.
- Do not invent custom branch prefixes.
- Do not use prefixes such as chore, refactor, hotfix, release, docs, test, ci, or task.
- Do not include ticket numbers unless they are explicitly part of the requested branch naming convention.
- Do not include file extensions, paths, underscores, spaces, punctuation, emojis, or uppercase letters.

---

# Branch Name Generator Skill

## Purpose

Generate a single Git branch name based on the provided project purpose and git diff.

The branch name must clearly describe the intent of the change while staying short, conventional, and easy to read.

## When to use this skill

Use this skill when the user provides a project description and a git diff, and expects a branch name for the changes shown in that diff.

## Required input

The user will provide:

### Project purpose

{{ "Project purpose" | input }}

### Git diff output

{{ "git diff output" | editor }}

## Output contract

The final answer must contain exactly one line and exactly one string.

Valid output forms are:

- `feature/[NAME]`
- `bugfix/[NAME]`

Where `[NAME]` must:

- contain 1 to 4 words
- use lowercase English words
- use hyphens between words
- use only ASCII letters and digits
- describe the actual change shown in the diff
- be concise but specific

Examples of valid branch names:

- `feature/user-auth`
- `feature/payment-retry`
- `feature/search-filters`
- `bugfix/login-redirect`
- `bugfix/token-refresh`
- `bugfix/empty-state`

These examples are illustrative only. Do not reuse them unless they accurately match the provided diff.

## Classification rules

Choose the branch prefix based on the primary intent of the diff.

### Use `feature/` when the diff primarily adds or expands behavior

Use `feature/` for changes such as:

- adding a new user-facing capability
- adding a new API endpoint
- adding a new UI component
- adding a new configuration option
- adding support for a new integration
- adding a new command, tool, route, page, or workflow
- expanding existing behavior in a meaningful way

### Use `bugfix/` when the diff primarily corrects broken behavior

Use `bugfix/` for changes such as:

- fixing incorrect logic
- fixing a runtime error
- fixing a regression
- fixing broken validation
- fixing incorrect UI behavior
- fixing incorrect API behavior
- fixing a crash, panic, exception, or failed edge case
- fixing tests because production behavior was wrong

### Mixed changes

If the diff contains both feature work and bug fixing:

1. Identify the dominant purpose of the diff.
2. If the diff mainly repairs existing behavior, use `bugfix/`.
3. If the diff mainly adds new behavior, use `feature/`.

### Refactors, docs, tests, and chores

Only `feature/` and `bugfix/` are allowed.

If the diff is mostly refactoring, documentation, tests, build changes, or internal cleanup:

- use `bugfix/` only if the change clearly fixes broken behavior
- otherwise use `feature/`
- choose a name that describes the meaningful outcome, not the mechanical action

For example, prefer names like:

- `feature/config-loader`
- `feature/request-validation`
- `bugfix/cache-invalidation`

Avoid vague names like:

- `feature/refactor`
- `feature/update`
- `feature/changes`
- `bugfix/fix`
- `bugfix/bug`

## Naming rules

When creating `[NAME]`:

1. Prefer domain-specific terms from the diff and project purpose.
2. Prefer nouns and short noun phrases.
3. Remove filler words such as `the`, `a`, `new`, `old`, `better`, `updated`, `changed`.
4. Avoid generic words unless they are domain-relevant.
5. Do not copy long function names directly unless they are the clearest domain concept.
6. Do not base the name only on filenames unless the filename represents the actual feature or bug.
7. Do not include implementation details unless they are the user-visible or domain-relevant point of the change.
8. Keep the name short enough to be readable in terminal output, pull requests, and CI logs.

## Decision process

Before answering:

1. Read the project purpose to understand the domain.
2. Read the git diff to identify what changed.
3. Determine whether the change is primarily a feature or a bug fix.
4. Extract 1 to 4 meaningful words that describe the change.
5. Convert the words to lowercase kebab-case.
6. Prefix the name with either `feature/` or `bugfix/`.
7. Validate the final result against the output contract.

## Validation checklist

Before returning the final answer, verify that:

- the answer is exactly one line
- the answer contains only one branch name
- the prefix is either `feature/` or `bugfix/`
- the name has 1 to 4 hyphen-separated words
- the name uses lowercase ASCII characters only
- the name contains no spaces
- the name contains no markdown formatting
- the name contains no explanation
- the name reflects the actual git diff
- no unsupported branch format was invented

## Final answer rule

Return only the branch name.

Do not explain your reasoning.
Do not include alternatives.
Do not include confidence.
Do not include punctuation after the branch name.
