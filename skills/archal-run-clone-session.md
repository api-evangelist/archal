---
name: Run an agent against an Archal clone session
description: >
  Provision a hosted, service-shaped clone session, drive the cloned service through the
  runtime proxy, then read the resulting trace and tear the session down. Use this to test
  an AI agent against GitHub/Slack/Stripe/etc. before it touches production.
api: openapi/archal-openapi-original.json
method: generated
generated: '2026-07-18'
operations:
  - listClones
  - createSession
  - getSession
  - proxyClonePost
  - proxyCloneGet
  - getTrace
  - deleteSession
---

# Run an agent against an Archal clone session

All control-plane calls send `Authorization: Bearer <archal token>` (a user token from
`archal login`, or a workspace key `archal_ws_<key>` via `ARCHAL_TOKEN`).

## Steps

1. **Discover clones** — `listClones` (`GET /api/clones`) to confirm the clone id you want
   (e.g. `github`, `slack`, `stripe`) is available.
2. **Create a session** — `createSession` (`POST /api/sessions`) with body
   `{ "clones": ["github"] }`. Send an `Idempotency-Key` header so retries do not create
   duplicate sessions. Optionally set `ttlSeconds`, `scenarioId`, or `seeds`. The response
   (`SessionSummary`) returns `sessionId`, plus `endpoints`, `apiBaseUrls`, and `mcp` maps.
3. **Poll status** — `getSession` (`GET /api/sessions/{sessionId}`) until `status` is
   `running` / `alive` is true.
4. **Drive the clone** — call the cloned service through the runtime proxy:
   `proxyClonePost` (`POST /runtime/{sessionId}/{cloneId}/api/{path}`) and
   `proxyCloneGet` (`GET /runtime/{sessionId}/{cloneId}/api/{path}`). Send the
   service-shaped `Authorization` header the clone expects AND the Archal
   `x-route-authorization` header for the outer hop. On `--docker`/`--sandbox` runs a
   bootstrap token is auto-injected if `Authorization` is absent.
5. **Read the trace** — `getTrace` (`GET /api/traces/{rootTraceId}`) for the tool calls,
   requests, state changes, and score.
6. **Tear down** — `deleteSession` (`DELETE /api/sessions/{sessionId}`) to stop the session
   (or let its TTL expire).

## Rules

- Idempotency: only `createSession` accepts `Idempotency-Key` (max 200 chars) — use it.
- Errors return the custom JSON envelope `{error, code, message, detail}`; the proxy also
  passes through the real clone's native error shape. `403` = not your resource, `426` =
  CLI upgrade required. See `errors/archal-problem-types.yml`.
- Clones never touch real services — safe to run destructive operations.
