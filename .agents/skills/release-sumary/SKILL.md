---
name: release-sumary
description: Build a weekly release-ready summary by matching Jira issues in ready statuses with GitLab merge requests authored by the current user.
---

# Release Summary

Use this skill when the user asks to prepare a weekly release summary for Jira and GitLab.

## Goal

Build a table of work items that may be included in a release, based on:
- Jira: a project key provided by the user at call time
- Jira statuses: only `PENDING` and `READY TO DEPLOY`
- Jira assignee: only issues assigned to the authenticated Jira user
- Jira sprint: only issues from the current sprint
- GitLab: merge requests authored by the current authenticated user
- Linking rule: text in the MR description of the form `Closes <PROJECT_KEY>-123`

## Core rules

1. Always answer in the same language as the user's request.
2. Write all instructions and internal reasoning for the LLM in English.
3. Produce the final answer in Russian.
4. Use Russian table headers in the final answer.
5. Require the Jira project key from the user at call time.
6. If the project key is missing, stop and ask the user for it before doing anything else.
7. If the user does not provide the project key yet, do not execute the skill logic.
8. Ignore all Jira issues outside statuses `PENDING` and `READY TO DEPLOY`.
9. Ignore all Jira issues not assigned to the authenticated Jira user.
10. Ignore all Jira issues not in the current sprint.
11. In GitLab, start with open merge requests authored by the authenticated user.
12. Also include up to 5 closed or merged authored merge requests that match one of the Jira issues from the first list.
13. Do not count the author as an approver.
14. If the data is incomplete or ambiguous, prefer to surface the anomaly rather than hide it.

## Input handling

The skill requires one user-provided value:
- `project key` — the Jira project key to inspect

Example invocation:
- `release-sumary PROJ`

If the user invokes the skill without a project key:
- ask for the project key in Russian
- do not continue
- on the next turn, rerun the skill with the provided value

## Workflow

### 1) Collect Jira issues ready for release

Query Jira for all issues in the provided project key that satisfy all of the following:
- status is `PENDING` or `READY TO DEPLOY`
- assignee is the authenticated Jira user
- issue is in the current sprint

Keep only these issues. Everything else is out of scope.

For each Jira issue, collect at least:
- key
- summary/title
- status
- URL

### 2) Collect GitLab merge requests authored by the current user

1. Determine the authenticated GitLab user.
2. Query all merge requests authored by that user.
3. Include all open merge requests.
4. Additionally include up to 5 closed or merged merge requests that reference one of the Jira issues selected in step 1.
5. Prefer the most recently updated closed or merged merge requests when applying the limit.
6. For each MR, inspect the description.
7. Find references of the form `Closes <PROJECT_KEY>-<number>`.

Treat the reference as the Jira link for that MR.

### 3) Build the mapping

Map Jira issues and merge requests by the closing reference in the MR description.

Matching rules:
- A single MR may reference multiple Jira issues.
- A single Jira issue may be referenced by multiple MRs.
- Do not collapse rows in a way that hides unmatched items.
- If there are multiple matches, create separate rows so that every Jira issue and every MR remains visible.

Always keep both lists complete:
- Jira issues with no MR must still appear.
- MRs with no matching Jira issue must still appear.

### 4) Check approvals

For each MR:
- collect the list of approvers
- exclude the MR author from approval count
- if approval data is unavailable, mark the item as needing review

## Table rules

Return a markdown table with the following columns exactly:

1. `Jira`
   - Markdown link to the Jira issue
   - If there is no Jira issue for the row, use `—`

2. `Gitlab`
   - Markdown link to the merge request
   - If there is no MR for the row, use `—`

3. `Статус`
   - Jira issue status
   - If there is no Jira issue, use `—`

4. `Апрувы`
   - approval count and approver usernames/accounts
   - format should make the count obvious, for example: `2: @alice, @bob`
   - if none, use `0`

5. `Комментарий`
   - `Готово к релизу` only when:
     - the Jira issue exists, and
     - there is a matching MR, and
     - the MR is not merged/closed, and
     - there is at least one approver other than the author
   - `Нужно проверить` when:
        - the Jira issue exists but there is no MR
        - the Jira issue exists but there are no valid approvals
        - the MR is merged or closed
        - the MR exists but there is no matching Jira issue
        - any other suspicious or incomplete case

## Output style

- The table itself should be the main answer.
- Keep extra commentary minimal.
- If you notice strange situations, add a short note after the table.
- Use markdown links for Jira and GitLab URLs.
- Keep the answer concise and release-focused.
- Table headers in the final answer must be in Russian.

## Suggested implementation approach for the assistant

1. If the project key is not provided, ask for it in Russian and stop.
2. Get the current authenticated Jira user and GitLab user.
3. Fetch Jira issues for the provided project key that are in the current sprint, have status `PENDING` or `READY TO DEPLOY`, and are assigned to the authenticated Jira user.
4. Fetch all open merge requests authored by the current user.
5. Fetch up to 5 additional closed or merged authored merge requests that match one of the selected Jira issues.
6. Parse MR descriptions for `Closes <PROJECT_KEY>-<number>` references.
7. Fetch approvals for each MR and remove self-approval from the count.
8. Build the final markdown table.
9. Add a short anomalies note if needed.
