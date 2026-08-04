---
name: Run a Gumloop flow and collect its output
description: Trigger a saved Gumloop flow via the API, poll it to completion, and read its outputs.
api: openapi/gumloop-openapi-original.yml
method: generated
generated: '2026-07-19'
operations: [getInputs, startFlow, getAutomationRun, killPipeline]
---

# Run a Gumloop flow

Use this to execute one of your saved Gumloop flows programmatically and retrieve its results.

## Auth
Send `Authorization: Bearer {api_key}` on every request (API keys require the Pro plan or above). Personal keys also send the user id via the `x-auth-key` header; team keys set `user_id` per request. Base URL: `https://api.gumloop.com/api/v1`.

## Steps
1. (Optional) `getInputs` — `GET /get_inputs` to retrieve the input schema for the `saved_item_id` so you know which Input node names to send.
2. `startFlow` — `POST /start_pipeline` with `user_id` and `saved_item_id`, plus any extra body keys (matched by name to Input nodes; the whole body is forwarded to a Webhook Input node). Capture the returned `run_id`.
3. `getAutomationRun` — `GET /get_pl_run?run_id=...&user_id=...` and poll until the run reaches a terminal state; read outputs from Output nodes.
4. `killPipeline` — `POST /kill_pipeline` to stop a run and its subflows if you need to abort.

## Rules
- Triggering is **not idempotent** (no Idempotency-Key); do not blindly retry `startFlow` — instead poll the existing `run_id`.
- Concurrency-limited: a `429` means you exceeded simultaneous-run limits (2/5/15 by plan). Wait for a run to finish, then retry; Enterprise queues automatically.
- Errors are plain JSON (not RFC 9457): `401` invalid key, `403` key/permission mismatch, `404` run/saved item not found. See errors/gumloop-problem-types.yml.
