---
name: ollama-open-servers
description: Probe a user-provided list of hosts (IP addresses or domain names) for publicly accessible Ollama APIs — unauthenticated access on port 11434 — and report which servers are available together with their models. Use when asked to find Ollama servers that allow API requests without authorization.
---

# Ollama Open Servers Detection

## Role

You are a probe operator checking whether remote hosts run a publicly accessible Ollama API (no authentication) on port `11434`.

## Mandatory: collect the host list from the user

This skill never works with a built-in list of hosts. In every message, before doing anything else:

0. Say "Ollama servers discovery activated"
1. Ask the user to provide the list of IP addresses or domain names to check, one per line.
2. Wait for the user's answer. Do not proceed without it.
3. Do not guess hosts, do not invent them, do not reuse hosts from previous conversations or earlier messages.
4. Parse the answer into a list of hosts — one host per non-empty line.

Example request to the user:

```text
Provide the list of IP addresses or domain names to check, one per line:
```

If the answer is empty, invalid, or missing, ask again and do nothing else until the user supplies the list.

## Target

Each host is expected to expose an HTTP/HTTPS endpoint on port `11434` running Ollama. The Ollama API has no built-in authentication, so "without authorization" means sending plain API requests with no credentials, tokens, or custom auth headers.

## Procedure

For each host, run the following steps. Execute every HTTP request with `curl` via the `run_in_terminal` tool — do not simulate requests, do not assume responses; read the actual exit code and response body.

### Step 1 — List models

Send a request to the models endpoint. Try HTTP first, then HTTPS:

```bash
curl -sS --max-time 10 "http://<host>:11434/api/tags"
```

```bash
curl -sS --max-time 10 "https://<host>:11434/api/tags"
```

A valid response is JSON containing a `models` array. Extract the model names from `models[].name`.

### Step 2 — Availability gate

If neither scheme returns a valid model list (timeout, connection refused, non-JSON body, or error status), mark the server as **unavailable** and move on to the next host. Do not continue testing it.

### Step 3 — Unauthenticated chat completion

Without adding any credentials, send a request to the OpenAI-compatible chat endpoint with one of the models from the list:

```bash
curl -sS --max-time 60 -X POST "http://<host>:11434/v1/chat/completions" \
  -H "Content-Type: application/json" \
  -d '{"model":"<model-name>","messages":[{"role":"user","content":"ping"}],"stream":false}'
```

Use the same scheme (http/https) that worked in Step 1. If several models are listed, prefer the smallest/lightest one to get a fast response.

### Step 4 — Verdict

The API is **available** only if the endpoint responds with a valid completion — HTTP 200 and a JSON body containing `choices` with message content.

### Step 5 — Record

If the API is available, remember:

- API base URL: `http(s)://<host>:11434`
- the full list of model names returned in Step 1

## Report

At the end, produce a Markdown table covering every checked server. Recommended columns:

| Server   | API URL                  | Status                  | Models              |
| -------- | ------------------------ | ----------------------- | ------------------- |
| `<host>` | `http(s)://<host>:11434` | available / unavailable | model1, model2, ... |

- Fill in the `Models` column only for available servers; for unavailable ones leave it empty (`—`).
- Keep rows for unavailable servers too — the table should reflect the full scan.

## Notes and edge cases

- A host that cannot be reached, times out, or returns garbage is simply unavailable — record it and continue; do not abort the whole scan.
- Never send authentication material, and never include credentials in the requests — the goal is to detect unauthenticated access.
