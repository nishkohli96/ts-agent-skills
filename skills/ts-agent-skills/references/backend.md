# Backend (Node / Express / NestJS)

## Structure

- Layers: route/controller (transport) → service (business logic) → repository (data access). Controllers stay thin.
- Dependency injection (NestJS native, or constructor injection in Express) for testability.
- Folder layout: see `folder-structure.md` (generic and NestJS variants).

## API design

- Consistent REST naming (plural nouns), correct status codes, one error envelope: `{ error: { code, message, details? } }`.
- Validate params, query and body (schema middleware, or Nest `ValidationPipe` with whitelisting and rejecting unknown fields).
- Paginate every list endpoint (cursor-based for large or real-time sets). Cap `limit`.
- Version APIs (`/v1`) once there are external consumers.
- Idempotency keys for payment- or order-creating POSTs.

## Outbound HTTP

- Native `fetch` (Node 18+) through a typed wrapper in `lib/http.ts`; timeouts via `AbortSignal`. No axios unless the repo already uses it.
- Validate third-party responses with a schema before use.

## Data

- Parameterized queries or ORM only; never string-concat SQL or build Mongo queries from raw input.
- Index every frequent filter/sort; check query plans for hot paths. Avoid N+1 (joins, `include`, DataLoader).
- Transactions for multi-write operations. Migrations checked in; never edit applied migrations.
- Select only needed fields; never return password hashes or internal fields (use response DTOs/serializers).

## Reliability

- Timeouts and retries with backoff on outbound calls; circuit-break flaky dependencies.
- Graceful shutdown: handle `SIGTERM`, stop accepting requests, drain, close DB/queue connections.
- Long work goes to queues/workers (SQS, BullMQ), not the request cycle.
- Health and readiness endpoints.

## Observability

- Structured JSON logging (pino) with request IDs; no `console.log` in production code.
- Never log secrets, tokens, passwords or full PII.
- Error tracking (Sentry/Datadog) and basic metrics on latency and error rate.
