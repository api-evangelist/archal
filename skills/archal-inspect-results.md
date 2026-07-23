---
name: Inspect Archal runs, traces, and results
description: >
  Report on agent evaluation results — enumerate active sessions, pull a canonical run
  record, page through trace roots, read a trace in detail, and prune old traces.
api: openapi/archal-openapi-original.json
method: generated
generated: '2026-07-18'
operations:
  - listSessions
  - getRun
  - listTraces
  - getTrace
  - ingestTrace
  - deleteTraces
---

# Inspect Archal runs, traces, and results

Authenticate every call with `Authorization: Bearer <archal token>`.

## Steps

1. **List active sessions** — `listSessions` (`GET /api/sessions`) to see sessions you own
   and their `sessionId`s.
2. **Get a run record** — `getRun` (`GET /api/runs/{runId}`) for the canonical record of a
   single scored run.
3. **Page trace roots** — `listTraces` (`GET /api/traces`) with `limit` (default 20),
   `since` (RFC 3339), `sessionId`, `scenario`, and `includeStats` to filter and roll up.
4. **Read a trace** — `getTrace` (`GET /api/traces/{rootTraceId}`) for the full record of
   tool calls, API requests, state changes, and scoring.
5. **Ingest a trace** (optional) — `ingestTrace` (`POST /api/traces`) to upload a trace
   payload (e.g. from CI or a production source for autoloop).
6. **Prune** — `deleteTraces` (`DELETE /api/traces`) with `rootTraceId`, `before`, or
   `all=true` to remove trace roots.

## Rules

- Pagination is cursor/time-window via `limit` + `since`; see `conventions/archal-conventions.yml`.
- `503` means a backing service is unavailable — retry with backoff.
- Telemetry/tracing is off by default; traces exist only when the run had tracing enabled.
