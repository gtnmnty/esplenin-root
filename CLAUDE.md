# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository layout

This folder holds one product in three parts. `hotel-reservation/` and `frontend/` are their own git repos. This root is also a git repo (no remote yet) that tracks the context files, project skills and docs.

- `hotel-reservation/` — Spring Boot 4.1.2 backend (Java 21, Maven wrapper). Its own git repo.
- `frontend/` — Next.js 16.4.6 + React 19 + TypeScript app, pnpm. Its own git repo. Has its own `AGENTS.md`.
- `context/` — the project specification and working rules (`architecture.md`, `code-standards.md`, `ai-workflow-rules.md`, `project-overview.md`, `progress-tracker.md`, `test-cases.md`, `ui-context.md`). Read `architecture.md` and `code-standards.md` before designing anything.
- `.claude/skills/` — project-local skills, run as slash commands (`/architect`, `/kickoff`, and others).

Current state: the backend has entities, enums, and a Flyway setup, but no controllers, services, or repositories yet. `SecurityConfig` and `JwtAuthenticationFilter` are empty stubs, and `db/migration/V1__init.sql` is empty. The frontend is still the default `create-next-app` page. Do not assume features exist because the docs describe them.

## Commands

### Backend (run from `hotel-reservation/`)

On Windows use `mvnw.cmd` in place of `./mvnw`.

- Start the app: `./mvnw spring-boot:run` (dev profile is the default)
- Build and run all tests: `./mvnw verify`
- Run one test class: `./mvnw test -Dtest=HotelReservationApplicationTests`
- Run one test method: `./mvnw test -Dtest=HotelReservationApplicationTests#methodName`
- Run the app with Testcontainers-backed services: `TestHotelReservationApplication` (main method in `src/test`)

Tests use Testcontainers, so Docker must be running. `TestcontainersConfiguration` starts `postgres:latest` and `redis:latest`.

Local services: `docker compose up -d` in `hotel-reservation/` starts Postgres 18, Redis 8.2, and Mailpit (SMTP on 1025, web UI on 8025). Spring Boot's Docker Compose support also starts these automatically in dev.

Configuration: `application.yaml` reads everything from environment variables. It also imports `.env` in the module directory if present. The `.env*` files contain secrets; do not print or copy them into code or chat. Profiles `staging` and `prod` switch on SMTP auth and STARTTLS.

### Frontend (run from `frontend/`)

- Install: `pnpm install` (the repo pins `pnpm@12.16.1` via `packageManager`)
- Dev server: `pnpm dev` (http://localhost:3000)
- Production build: `pnpm build`
- Lint: `pnpm lint`

There is no `test` script in `package.json` yet, so there is no frontend unit or E2E test command. The docs mention Vitest and Playwright, but neither is installed.

Next.js 16 has breaking changes from earlier versions. Before writing frontend code, read the relevant guide in `frontend/node_modules/next/dist/docs/` as `frontend/AGENTS.md` instructs.

## Architecture

### Request path

Browser → Vercel (Next.js, forwards `/api/*`) → NGINX → Spring Boot API → PostgreSQL (Neon in staging/prod), Redis, Stripe, Cloudinary, SMTP. Stripe webhooks go directly to the API through NGINX. The frontend forwards `/api/*` to the backend, adding the real client IP header and a shared secret (`CLIENT_IP_HEADER_NAME`, `CLIENT_IP_SHARED_SECRET`). The same header name must match in the frontend, NGINX, and backend.

### Backend structure

Feature packages under `com.esplenin.hotel_reservation`: `auth`, `branches`, `rooms`, `availability`, `reservations`, `payments`, `billing`, `notifications`, `reports`, `audit`, `admin`. Each uses controller → service → repository. Only `config/` and `models/` exist today. Create new feature packages by following that layering; confirm before adding new top-level folders.

- `models/entities/` — JPA entities. Most extend `BaseEntity`, `TimestampedEntity`, `CreatedEntity`, or `VersionedEntity`.
- `models/enums/` — status and type enums. The docs say statuses and types are stored as text with CHECK constraints, not as database enums.
- Entities must never be returned from an API. Map them to DTOs with MapStruct.

### Access control

Roles: `RECEPTIONIST`, `SUPERVISOR`, `MANAGER`, `SUPER_ADMIN`, plus unauthenticated guests who use private tokens instead of accounts. Authorization must be enforced on the server for every request, with method-level security and ownership checks before any mutation. The frontend's route guard (`proxy.ts`, not `middleware.ts`) is only a convenience. Branch-scoped roles see only their own branch; `SUPER_ADMIN` sees all.

Auth: JWT access tokens (15 min) and rotating refresh tokens (7 days, stored hashed, reuse ends the session family), both in httpOnly cookies. Invitation and reset tokens are single-purpose and stored hashed. CSRF protection is required for state-changing requests.

### Invariants (from `context/architecture.md`)

These must hold in any change, and the docs list the full set:

- A room type is never sold more than its room count. Inventory is claimed per night with a guarded update in date order.
- Two active reservations cannot overlap on the same room (exclusion constraint).
- Reservations and payments are never deleted, only cancelled or voided. The audit log is append-only.
- The server computes prices, taxes, totals, and refunds. Never trust client values.
- Raw card data (number, expiry, CVV) must never reach the server, logs, or the database. Stripe holds card data.
- A card reservation is created only after payment succeeds. Booking details live in a checkout intent until then.
- Stripe webhooks: verify the signature, process each event once, and use idempotency keys on payment calls.
- Money is stored as integer minor units plus a currency code. Dates are branch-timezone dates; timestamps are UTC.
- Taxes and prices are copied onto the reservation at booking time so old bookings do not change.
- Tokens never appear in logs, Sentry, or PostHog.
- Schema changes only through Flyway migrations, forward only. Never edit a migration that has already run. Neon: the app uses the pooled connection, Flyway the direct one.
- Every reservation starts `Pending`; statuses are Pending, Approved, Cancelled, No Show, and the last two are final. A receptionist assigns the room.
- Refunds are all automatic (no cancellation or no-show fees). Only goodwill refunds need approval, within the Supervisor limit, then Manager or Super Admin.
- Guest, invitation, reset and history tokens are stored hashed and single-purpose. Invitation tokens are used up when the password is submitted, not on page load.
- Checkout must not depend on Redis.

### Schema and spec

The authoritative schema is `hotel_schema_v4.1.md` in this folder (39 tables). It supersedes v3.1 and was written to follow spec v0.3, plus the v4.1 review fixes listed at the top of the file. The project documentation v0.4 matches it. If they ever differ on tables or columns, the schema file wins. It must not change until the project owner says so. Behavior comes from the project documentation (S02 scope items, S08 default values). If a requirement is unclear, record it as an open question in `context/progress-tracker.md` instead of guessing.

## Conventions

- Backend API: base path `/api/v1`, JSON only. Lists use `page` and `size` with a maximum size. Payment and booking creation accept an idempotency key. Every response carries a request ID that also appears in logs.
- Errors: one exception handler producing RFC 7807 problem details with stable error codes.
- Use an injected `Clock` for time logic so tests can move time. Use optimistic locking (`@Version`) on records edited concurrently.
- Frontend: TypeScript strict mode, React Hook Form + Zod for forms, TanStack Query for data fetching, next-intl for translations (English, French, Japanese, Chinese). Server components by default.
- Styling uses Tailwind CSS tokens from `context/ui-context.md`. Do not hard-code colors. Do not hand-edit `components/ui/` (generated by the shadcn CLI).
- Do not hard-code limits or durations from the spec; read them from the S08 defaults or branch settings.
- Git: commit format `type(scope): short summary` with types `feat, fix, docs, refactor, test, chore, ci`, imperative mood, 72 characters or less. Branch names use `feature/...`, `fix/...`, `docs/...`, `chore/...`. The `main` branch is protected.

## Working rules

Full rules are in `context/ai-workflow-rules.md`. The ones that matter most day to day:

- Source of truth is `Esplenin_Hotel_Project_Docs_v0.4.docx` (root); the `context/` files summarize it. When sources disagree: numbers and limits come from S08, behavior and permissions from S02 then S03 then S08 rules, pages from S04, technology from S05, tables from the schema file. S02 is the scope contract, so update it before changing scope.
- Start a session by reading `context/progress-tracker.md`. Build order follows S14 phases (Phase 0 Foundation first; core booking flow before cache, rate limiting, NGINX and monitoring).
- A unit is done only when its `Must` cases in `context/test-cases.md` pass, access rules are checked for every role and branch, audit entries and translatable texts exist, and the tracker is updated. Run `./mvnw verify` (backend) and `pnpm build` plus `pnpm lint` (frontend).
- Project skills in `.claude/skills/<name>/SKILL.md` (`/architect`, `/migrate`, `/trace`, `/review`, `/docs-sync`, and others) define the session routine. They run as slash commands.
- One feature per branch; for cross-repo features use the same branch name in both repos and merge the backend first. PRs list the S02 scope numbers they cover.

- Work on one feature unit at a time. Split changes that mix backend and frontend, migrations and business logic, or payments and UI.
- Do not invent product behavior that the spec does not define.
- Do not modify `.env` files, lock files, or `components/ui/*`.
- Update `context/progress-tracker.md` and the relevant context file when architecture, conventions, or scope change.
