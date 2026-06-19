---
name: resolve-input-placeholders
description: Detect {{ "..." | input }} placeholders in task instructions. Before starting work or calling any tools, collect every required value from the user, resolve all placeholders, and only then execute the task. Use whenever a prompt contains input placeholders.
---
# Resolve Input Placeholders Before Execution

## Purpose

Some task instructions contain input placeholders that require information from the user.

An input placeholder has this form:

```text
{{ "Question for the user" | input }}
```

Example:

```text
{{ "How to run tests" | input }}
```

The text inside quotation marks is the question that must be asked to the user.

The `input` operator means that the value must be provided by the user before the task can begin.

## Mandatory behavior

Before starting any task:

1. Read the entire task.

2. Find every input placeholder matching this pattern:

   ```text
   {{ "..." | input }}
   ```

3. Extract the text inside quotation marks from every placeholder.

4. Ask the user for the required values.

5. Wait until the user has answered every question.

6. Replace each placeholder with the corresponding user-provided value.

7. Only after all placeholders are resolved, begin executing the task.

## Strict rules

* Do not begin the task while any input placeholder remains unresolved.
* Do not call tools before all required input values are collected.
* Do not edit files, run commands, inspect the project, or make implementation decisions before collecting the required values.
* Do not guess or infer placeholder values.
* Do not replace placeholders with defaults unless the user explicitly provides those defaults.
* Do not treat placeholder text as the actual value.
* Do not skip a placeholder because its answer appears obvious.
* If the user's answer is incomplete, ask only for the missing values.
* If the same placeholder question appears multiple times, ask it only once and reuse the answer.
* Preserve the order in which placeholders appear in the task.
* When multiple placeholders exist, ask all questions in one numbered list.
* An explicit answer such as `none`, `skip`, an empty value, or `not applicable` is valid if the user intentionally provides it.

## Required response when placeholders are found

Ask only for the missing input values.

Example task:

```markdown
Implement the requested change.

- Run tests: {{ "How to run tests" | input }}
- Run linters: {{ "How to run linters" | input }}
```

Correct response:

```text
Before I begin, I need the following values:

1. How to run tests?
2. How to run linters?
```

Do not begin implementation in the same response.

## After receiving the answers

Confirm that every placeholder has a value.

Internally resolve the task as if the placeholders had been replaced with the user's answers.

For example:

```text
{{ "How to run tests" | input }}
```

with the user answer:

```text
cargo test
```

must be treated as:

```text
cargo test
```

Then begin the task normally.

## No-placeholder case

If the task contains no input placeholders, proceed immediately without asking additional questions.

## Completion condition

Placeholder resolution is complete only when:

* every input placeholder has been detected;
* every unique placeholder question has been answered;
* no unresolved `input` placeholder remains;
* task execution has not started prematurely.
