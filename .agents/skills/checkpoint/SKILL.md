---
name: work-checkpoint
description: Save, update, and restore reliable checkpoints for long-running software development tasks. Use when work reaches a meaningful milestone, before risky changes, before switching models or sessions, when context may be lost, or when the user explicitly asks to preserve the current state.
---

# Work Checkpoint Skill

## Purpose

Preserve enough verified context for another agent or a future session to resume the current development task safely and efficiently.

A checkpoint is not a transcript and not a replacement for Git. It records the semantic state of the work:

- the current goal;
- what has been completed;
- important decisions and their reasons;
- changed areas of the repository;
- validation already performed;
- known failures, risks, and uncertainties;
- the next concrete steps.

The repository, Git state, files, and test results remain the source of truth.

## When to use this skill

Create or update a checkpoint when any of the following is true:

- the user explicitly asks to save, preserve, checkpoint, or hand off the current work;
- a meaningful implementation milestone has been completed;
- an important architectural or behavioral decision has been made;
- a long debugging session produced useful findings;
- the next step is risky, destructive, or difficult to reverse;
- the current session or model may be replaced;
- available context is becoming limited;
- work is about to stop before the task is finished.

Do not update the checkpoint after every trivial edit. The checkpoint should change only when the information needed to resume the task has materially changed.

## When not to use this skill

Do not create a checkpoint for:

- a tiny task that is already complete and obvious from the final diff;
- purely conversational discussion with no repository work;
- speculative ideas that were not accepted or implemented;
- information already documented accurately in a canonical project document;
- temporary thoughts that have not been verified.

## Storage layout

Use this project-local layout by default:

```text
.agents/
  checkpoint.md
  checkpoints/
```

Use `.agents/checkpoint.md` as the latest resumable state.

Create an archived milestone snapshot only when the checkpoint represents a major stage:

```text
.agents/checkpoints/YYYY-MM-DDTHH-mm-ssZ-short-label.md
```

Examples:

```text
.agents/checkpoints/2026-07-10T11-30-00Z-auth-refactor-complete.md
.agents/checkpoints/2026-07-10T14-05-00Z-before-schema-migration.md
```

If the repository already has an established location for agent state, follow the repository convention instead of creating a competing structure.

Do not add `.agents/` to Git, remove it from Git, or modify `.gitignore` unless the user explicitly requests that behavior or the repository instructions already define it.

## Core rules

### 1. Verify before writing

Before creating or updating a checkpoint, inspect the current repository state when tools are available:

```bash
git status --short
git diff --stat
git diff --cached --stat
```

Inspect relevant files and recent changes. Run or review the most relevant validation commands when practical.

Never describe an implementation as complete solely because it was intended or discussed.

### 2. Separate facts from uncertainty

Write verified facts as facts.

Mark unverified statements explicitly using labels such as:

- `Unverified:`
- `Assumption:`
- `Suspected:`
- `Needs confirmation:`

Do not convert a hypothesis into a conclusion while summarizing.

### 3. Do not save chain-of-thought

Do not record private reasoning, hidden deliberation, internal scratch work, or a chronological transcript of the conversation.

Record only concise, useful artifacts of reasoning:

- decisions;
- rationale;
- rejected alternatives;
- evidence;
- constraints;
- unresolved questions.

### 4. Do not duplicate source code or diffs

Do not paste large code blocks, complete files, or large diffs into the checkpoint.

Refer to files, symbols, commits, commands, and small critical snippets instead.

Good:

```text
`src/auth/session.ts`: refresh retry logic now rejects reused refresh tokens.
```

Bad:

```text
A 400-line copy of `src/auth/session.ts`.
```

### 5. Never fabricate validation

For every validation command, record one of:

- `Passed`
- `Failed`
- `Not run`
- `Partially run`

Include the relevant failure summary when a command fails.

Never write that tests passed when they were not run.

### 6. Preserve exact technical identifiers

Do not abbreviate or paraphrase:

- file paths;
- function, type, class, and variable names;
- package names;
- commands;
- environment variable names;
- API routes;
- schema fields;
- error messages when they are relevant.

### 7. Keep the checkpoint resumable

A new agent should be able to answer these questions after reading the checkpoint:

1. What is the user trying to achieve?
2. What is the current verified state?
3. What changed?
4. Why were the important decisions made?
5. What remains unfinished?
6. What should be done next?
7. What must be checked before modifying anything else?

If the checkpoint cannot answer these questions, it is incomplete.

### 8. Do not perform unrelated actions

Saving a checkpoint does not authorize the agent to:

- create a Git commit;
- push changes;
- amend history;
- create or switch branches;
- modify `.gitignore`;
- delete temporary files;
- run destructive commands.

Perform those actions only when separately authorized.

## Checkpoint creation workflow

When saving the current state:

1. Read repository instructions such as `AGENTS.md`, `CONTRIBUTING.md`, or project-specific agent rules.
2. Inspect Git status and relevant diffs.
3. Identify the current task and accepted scope.
4. Inspect the files that contain the current implementation.
5. Review validation results or run the smallest relevant checks when practical.
6. Distinguish completed work from planned work.
7. Write `.agents/checkpoint.md` using the required template.
8. Re-read the file and remove stale, speculative, or redundant statements.
9. Create an archived snapshot only for a meaningful milestone.
10. Report what was saved and clearly mention any validation that was not performed.

Write the file atomically when possible: create a temporary file and replace the old checkpoint only after the new content is complete.

## Resume workflow

When resuming from a checkpoint:

1. Read `.agents/checkpoint.md`.
2. Read repository instructions.
3. Inspect current Git status and diff.
4. Inspect every file named in `Changed areas` and `Next steps` that is relevant to the next action.
5. Compare the checkpoint with the current repository state.
6. Re-run the smallest important validation commands when practical.
7. Report any mismatch before continuing.
8. Treat the repository and test results as authoritative when they conflict with the checkpoint.
9. Continue from the first valid incomplete step, not blindly from the prose.

Do not assume the checkpoint is current merely because it exists.

## Required checkpoint format

Use the following structure for `.agents/checkpoint.md`.

```markdown
# Work checkpoint

## Metadata

- Updated: `<ISO 8601 timestamp with timezone>`
- Repository: `<repository name or path>`
- Branch: `<current branch or unknown>`
- HEAD: `<commit hash or unknown>`
- Working tree: `<clean | modified | conflicted | unknown>`
- Task status: `<in progress | blocked | ready for review | complete>`

## Current task

A concise description of the user's actual goal and accepted scope.

## Current verified state

- What is currently implemented.
- What behavior is confirmed.
- What remains incomplete.

## Decisions

### `<decision title>`

- Decision: `<what was chosen>`
- Reason: `<why it was chosen>`
- Alternatives considered: `<relevant rejected options, or none>`
- Constraints: `<important constraints, or none>`

## Changed areas

- `<path or symbol>`: `<what changed and current state>`

## Validation

| Command or check | Result | Notes |
|---|---|---|
| `<command>` | `<Passed | Failed | Not run | Partially run>` | `<important details>` |

## Known issues and risks

- `<verified issue, risk, assumption, or uncertainty>`

Use explicit labels where appropriate:

- `Unverified:`
- `Assumption:`
- `Suspected:`
- `Needs confirmation:`

## Next steps

1. `<single concrete next action>`
2. `<next action after that>`
3. `<optional later action>`

Each step must describe an observable action or result. Avoid vague instructions such as "continue implementation" or "finish the task".

## Resume checklist

- [ ] Read repository instructions.
- [ ] Inspect `git status --short`.
- [ ] Inspect current diff.
- [ ] Verify checkpoint against named files.
- [ ] Re-run the most relevant validation.
- [ ] Resolve any mismatch before editing.

## Handoff note

A short note containing anything the next agent must not overlook.
```

## Quality requirements

A good checkpoint is:

- concise enough to scan quickly;
- specific enough to resume without the original conversation;
- grounded in repository state;
- explicit about uncertainty;
- clear about what was and was not validated;
- focused on the current task;
- free of conversational filler.

A bad checkpoint:

- summarizes the entire chat;
- claims success without evidence;
- contains stale plans as if they were implemented;
- copies large source files;
- hides failing tests;
- says only "continue from here";
- uses vague references such as "the auth file" when an exact path exists.

## Example: good checkpoint entry

```markdown
## Current verified state

- `POST /sessions/refresh` now rejects a refresh token after its first successful use.
- Token rotation is implemented in `internal/auth/refresh.go`.
- The database migration for the `used_at` column exists but has not been applied locally.
- The handler test for concurrent token reuse still fails intermittently.

## Validation

| Command or check | Result | Notes |
|---|---|---|
| `go test ./internal/auth/...` | Failed | `TestConcurrentRefreshReuse` failed 1 of 20 runs. |
| `go test ./internal/http/...` | Passed | No failures. |
| Migration applied locally | Not run | Local database was unavailable. |

## Next steps

1. Reproduce `TestConcurrentRefreshReuse` with `go test -run TestConcurrentRefreshReuse -count=100 ./internal/auth/...`.
2. Verify that token invalidation and replacement occur in the same database transaction.
3. Apply the migration locally and rerun the auth and HTTP test suites.
```

## Example: bad checkpoint entry

```markdown
Everything is mostly done. I changed auth and tests. There may be a race somewhere. Continue debugging and run tests.
```

This is not resumable because it lacks exact paths, verified state, validation results, and concrete next actions.

## User-facing completion message

After saving a checkpoint, report only the essential result:

- where the checkpoint was saved;
- whether an archive snapshot was created;
- the current task status;
- which validation was performed;
- any important failure or unverified assumption.

Do not paste the entire checkpoint into the response unless the user asks to see it.
