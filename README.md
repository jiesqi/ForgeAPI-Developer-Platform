# ForgeAPI

ForgeAPI is a developer-focused API control room for managing projects, credentials, request traffic, usage analytics, and webhook endpoints. The current implementation uses the workspace's TypeScript service stack so it can run directly in this project with real PostgreSQL persistence and a generated OpenAPI client.

## What is implemented

- Dashboard summary with live request, latency, success-rate, and endpoint metrics
- Project CRUD with status, environment, search, filtering, and pagination
- API key creation with one-time secret display and revocation
- Request log search, status filtering, and pagination
- Usage analytics for daily traffic, methods, status classes, and top endpoints
- Webhook CRUD with event subscriptions and one-time signing-secret display
- Generated React Query client and Zod schemas from `lib/api-spec/openapi.yaml`
- Seeded development data for a useful first run
- Structured server logging with request metadata

Authentication, distributed rate limiting, asynchronous delivery workers, and the remaining production controls are tracked in the roadmap rather than represented as finished functionality.

## Stack

- React + Vite dashboard
- Express API server
- PostgreSQL with Drizzle ORM
- OpenAPI 3.1 contract-first code generation
- Zod validation
- pnpm workspaces

## Requirements

- Node.js 24+
- pnpm
- PostgreSQL

## Local setup

```bash
pnpm install
cp .env.example .env
```

Set `DATABASE_URL` to a PostgreSQL connection string. The Replit workspace already provides this variable in development.

Push the schema and start the services:

```bash
pnpm --filter @workspace/db run push
pnpm --filter @workspace/api-server run dev
pnpm --filter @workspace/forge-api run dev
```

The API is served under `/api`, and the dashboard is served at `/`.

## Contract-first API

The source of truth is `lib/api-spec/openapi.yaml`. Regenerate the typed client after changing it:

```bash
pnpm --filter @workspace/api-spec run codegen
```

Example:

```bash
curl http://localhost:80/api/v1/projects
curl http://localhost:80/api/v1/dashboard
```

API responses for collection endpoints include a `data` array and pagination metadata. API key and webhook secrets are only returned during their creation response and are stored as hashes.

## Development checks

```bash
pnpm run typecheck
pnpm --filter @workspace/forge-api run build
```

## Project structure

- `artifacts/forge-api` — dashboard application
- `artifacts/api-server` — Express API routes and server bootstrap
- `lib/api-spec` — OpenAPI source and codegen command
- `lib/api-client-react` — generated React Query client
- `lib/api-zod` — generated server-side validation schemas
- `lib/db` — Drizzle schema and database client
- `docs` — architecture and API usage notes

## Security notes

Do not commit `.env` files or secrets. Request logs deliberately exclude authorization headers and API secrets. The development seed uses clearly labeled demo credentials and stores only deterministic hashes.

## Roadmap

1. Replit Auth session integration and user/team ownership
2. API key authentication, scopes, rotation, and expiry enforcement
3. Redis-backed rate limiting with standard rate-limit headers
4. Sidekiq-equivalent background delivery workers and webhook retries
5. Admin authorization, audit events, and production deployment hardening