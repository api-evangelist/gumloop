---
name: Create a Gumloop agent and run a session
description: Create an agent, open a session, send messages, and read the agent's replies.
api: openapi/gumloop-openapi-original.yml
method: generated
generated: '2026-07-19'
operations: [createAgent, createSession, retrieveSession, sendMessage, cancelSession]
---

# Run a Gumloop agent session

Use this to drive a conversational Gumloop agent from your own application.

## Auth
`Authorization: Bearer {api_key}` on every request. Base URL: `https://api.gumloop.com/api/v1`.

## Steps
1. `createAgent` — `POST /agents` to create an agent on a team you have create permission for (or reuse an existing `agent_id` from `listAgents`).
2. `createSession` — `POST /agents/{agent_id}/sessions`. With an `input`, the message is enqueued and the response is `202` with state `processing`/`queued`; without `input` you get a `201` idle stub.
3. `retrieveSession` — `GET /sessions/{session_id}` to read messages, current state, and participants. Poll until the session reaches a terminal state (`idle`, `completed`, or `failed`).
4. `sendMessage` — `POST /sessions/{session_id}/messages` to append a user message and resume the agent. The session must be terminal first, or you get `409 interaction_not_in_terminal_state`.
5. `cancelSession` — `POST /sessions/{session_id}/cancel` to abort a `processing`/`queued` session.

## Rules
- A `409` on `sendMessage` means the session is still `processing`/`queued` — wait for terminal state.
- Sessions are cursor-paginated (`listSessions`), newest first; use `page_size` (integer) and the returned cursor.
- Concurrency-limited (agent interactions: 5/25/100 by plan); a `429` means the limit was exceeded.
