---
name: Discover and call MCP tools through Gumloop
description: List the MCP servers in your Gumloop catalog, inspect their tools, and execute tool calls.
api: openapi/gumloop-openapi-original.yml
method: generated
generated: '2026-07-19'
operations: [listMcpServers, retrieveMcpServer, listMcpServerTools, callMcpTools]
---

# Call MCP tools through Gumloop

Use this to invoke tools on the MCP servers connected to your Gumloop account (Gumloop-hosted `gumcp_server`, user-deployed `gumstack_server`, and custom `mcp_server`).

## Auth
`Authorization: Bearer {api_key}` on every request. Base URL: `https://api.gumloop.com/api/v1`.

## Steps
1. `listMcpServers` — `GET /mcp/servers` to see the servers visible to you and each one's connection state.
2. `retrieveMcpServer` — `GET /mcp/servers/{server_id}`; the response's `allowed_tool_call_ids` are the tool calls you may invoke on that server.
3. `listMcpServerTools` — `GET /mcp/servers/{server_id}/tools`. If the server is not `connected`, `tools` is empty and a `gumloop_auth_url` is returned so a user can authenticate.
4. `callMcpTools` — `POST /mcp/tools/call` with a batch of 1–5 tool calls. Calls run concurrently and each result carries its own `status`; MCP-level failures (auth, policy block, invalid tool, upstream HTTP error) come back in `results[]` rather than as a top-level error.

## Rules
- Batch size is 1–5; fewer than 1 or more than 5 calls returns `400`.
- `404 mcp_server_not_found` means the server id is not in your catalog; `409 ambiguous_mcp_server` means duplicate entries — remove the duplicate.
- If a server is not connected, prompt the user through the returned `gumloop_auth_url` before calling tools.
