---
name: ollama-open-servers
description: Probe a user-provided list of hosts (IP addresses or domain names) for publicly accessible Ollama APIs — unauthenticated access on port 11434 — and report which servers are available together with their models. Use when asked to find Ollama servers that allow API requests without authorization.
---

# Ollama Open Servers Detection

## Role

You are a probe operator checking whether remote hosts run a publicly accessible Ollama API (no authentication) on port `11434`.

## Working mode: step by step with a checkpoint file

This scan can take a long time (many hosts, `--max-time` up to 60 s per request). Work strictly step by step and persist progress to a checkpoint file **after every step**. If the run is interrupted, the next invocation reads the checkpoint and continues only with the remaining hosts — finished requests are never repeated.

- Checkpoint path (constant): `.ollama-scan.tmp.md` in the workspace root. A dedicated name is used instead of a generic `.tmp.md` so the file cannot collide with temporary files of other tools.
- The checkpoint is Markdown: a counts line plus one table row per host. See "Checkpoint file format" below.
- Never commit the checkpoint file to version control.

## Phase 0 — Resume or collect the host list

This skill never works with a built-in list of hosts.

1. Say "Ollama servers discovery activated".
2. Check whether `.ollama-scan.tmp.md` exists.
   - If it exists and contains at least one row with status `pending`: tell the user `Resuming from checkpoint: X checked, Y remaining`, load the table, and go straight to Phase 2. Do not ask for a new host list.
   - Otherwise, ask the user to provide the list of IP addresses or domain names to check, one per line.
3. Wait for the user's answer. Do not proceed without it.
4. Do not guess hosts, do not invent them, do not reuse hosts from previous conversations or earlier messages.
5. Parse the answer into a list of hosts — one host per non-empty line, duplicates removed (keep the first occurrence).

Example request to the user:

```text
Provide the list of IP addresses or domain names to check, one per line:
```

If the answer is empty, invalid, or missing, ask again and do nothing else until the user supplies the list.

If the user explicitly supplies a new host list while a checkpoint with pending hosts already exists, treat it as a fresh scan: confirm with the user, then overwrite the checkpoint (Phase 1).

## Phase 1 — Initialize the checkpoint

After the host list is parsed:

1. Create/overwrite `.ollama-scan.tmp.md`: a counts line (`Total: N | Checked: 0 | Remaining: N`) and one row per host with status `pending`.
2. Confirm to the user: `Checkpoint saved to .ollama-scan.tmp.md — N hosts to check.`

## Phase 2 — Scan hosts, one at a time

Loop until no `pending` row remains in the checkpoint:

1. Pick the first host with status `pending` (in file order).
2. Run the per-host steps below (List models → Availability gate → Chat completion → Verdict).
3. Immediately after the verdict, rewrite the whole checkpoint file with the result (status, API URL, models, timestamp). **Do not start the next host before the checkpoint is updated.** If execution is interrupted at any point, at most the current host is lost.

### Per-host Step 1 — List models

Send a request to the models endpoint. Try HTTP first, then HTTPS:

```bash
curl -sS --max-time 10 "http://<host>:11434/api/tags"
```

```bash
curl -sS --max-time 10 "https://<host>:11434/api/tags"
```

A valid response is JSON containing a `models` array. Extract the model names from `models[].name`.

### Per-host Step 2 — Availability gate

If neither scheme returns a valid model list (timeout, connection refused, non-JSON body, or error status), mark the server as **unavailable** and move on to the next host. Do not continue testing it.

### Per-host Step 3 — Unauthenticated chat completion

Without adding any credentials, send a request to the OpenAI-compatible chat endpoint with one of the models from the list:

```bash
curl -sS --max-time 60 -X POST "http://<host>:11434/v1/chat/completions" \
  -H "Content-Type: application/json" \
  -d '{"model":"<model-name>","messages":[{"role":"user","content":"ping"}],"stream":false}'
```

Use the same scheme (http/https) that worked in Step 1. If several models are listed, prefer the smallest/lightest one to get a fast response. Check only one model per server.

### Per-host Step 4 — Verdict

The API is **available** only if the endpoint responds with a valid completion — HTTP 200 and a JSON body containing `choices` with message content.

### Per-host Step 5 — Record in the checkpoint

Update the host's row in `.ollama-scan.tmp.md`:

- Status: `available` or `unavailable` (never leave a checked host as `pending`).
- API URL: `http(s)://<host>:11434` — only for available servers, otherwise `—`.
- Models: the full list from Step 1 — only for available servers, otherwise `—`.
- Checked at: current UTC timestamp.
- Recompute the counts line (`Checked` / `Remaining`).

Rewrite the file in full after every host.

## Phase 3 — Final report

When all rows are `available` or `unavailable` (no `pending` left):

1. Read the checkpoint and produce the Markdown report table covering every checked server.
2. After presenting the report, delete `.ollama-scan.tmp.md` and mention that the progress file was removed. If the user asks to keep it, leave it in place and point to its path.

## Checkpoint file format

Example:

```markdown
# Ollama scan checkpoint — updated 2026-05-06T12:00:00Z
Total: 3 | Checked: 1 | Remaining: 2

| Server | API URL | Status | Models | Checked at |
| ------ | ------- | ------ | ------ | ---------- |
| host1  | http://host1:11434 | available | llama3.2:1b | 2026-05-06T12:00:00Z |
| host2  | —                  | pending   | —          | —          |
| host3  | —                  | pending   | —          | —          |
```

Rules:

- Status is one of `pending` / `available` / `unavailable`.
- `—` means "not known yet".
- Keep the row order stable; only edit the row being processed plus the counts line.
- The counts line is advisory — when resuming, trust the rows, not the counts.

## Report

At the end, produce a Markdown table covering every checked server. Recommended columns:

| Server   | API URL                  | Status                  | Models              |
| -------- | ------------------------ | ----------------------- | ------------------- |
| `<host>` | `http(s)://<host>:11434` | available / unavailable | model1, model2, ... |

- Fill in the `Models` column only for available servers; for unavailable ones leave it empty (`—`).
- Keep rows for unavailable servers too — the table should reflect the full scan.

## Notes and edge cases

- A host that cannot be reached, times out, or returns garbage is simply unavailable — record it and continue; do not abort the whole scan.
- Execute every HTTP request with `curl` using the shell tool — do not simulate requests, do not assume responses; read the actual exit code and response body.
- Never send authentication material, and never include credentials in the requests — the goal is to detect unauthenticated access.
- The checkpoint file is the single source of truth for what has been done: on resume, only `pending` rows are re-tested, and recorded verdicts are never re-requested.
