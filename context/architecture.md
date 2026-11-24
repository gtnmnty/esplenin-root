# Architecture Context

Source: Esplenin Hotel Project Documentation v0.4 (S05, S06, S07, S08, S09, S12). Numbers and limits live in S08 Default Values.

## Stack

| Layer | Technology | Role |
| --- | --- | --- |
| Backend | Spring Boot 4.1.x, Java 21 | All business rules, security checks, payment logic, background jobs |
| Frontend | Next.js 16.x + TypeScript, Node 24 | All pages, role route guards (`proxy.ts`), forwards `/api/*` to the backend |
| UI | Tailwind CSS + shadcn/ui, Recharts | Styling, components, dashboard charts |
| Database | PostgreSQL 18 on Neon | Data, including checkout intents |
| ORM / migrations | Spring Data JPA (Hibernate) + Flyway | Data access; schema changes only through migrations |
| Auth | Spring Security + jjwt, salted BCrypt | Staff sign-in, RBAC, JWT access + rotating refresh tokens |
| Payments | Stripe (test mode) | Cards, refunds, disputes, webhooks |
| Images | Cloudinary | Room photos (public), bank transfer receipts (private) |
| Cache / limits | Redis 8.2.x, Caffeine, Bucket4j + Resilience4j | Shared cache, rate-limit counters, local cache, circuit breaker |
| Edge | NGINX on Oracle | TLS, load balancer, first rate limit, static/public cache |
| Email | Spring Boot Mail + Thymeleaf; Mailpit (dev), Brevo (staging/prod) | Queued, retried, localized emails |
| Jobs | Spring `@Scheduled` + ShedLock + `@Async` | One instance runs each job |
| Mapping / docs | MapStruct, Lombok, springdoc-openapi | DTO mapping, boilerplate, Swagger UI |
| Frontend data | TanStack Query, React Hook Form + Zod, next-intl | Fetching, forms, translations |
| PDF | OpenPDF / Flying Saucer | Invoices, receipts, reports (PDF only) |
| Observability | Micrometer, Prometheus, Grafana, Logback JSON, Alloy, Loki, Sentry/GlitchTip, Uptime Kuma | Build last |
| Testing | JUnit 5, Mockito, Testcontainers, Vitest, Playwright, k6 | Unit, integration, E2E, load |
| CI/CD | GitHub Actions, self-hosted runner on Oracle, Docker Compose | Build, test, deploy |
| Hosting | Vercel Hobby (frontend), Oracle Always Free (backend, NGINX, Redis), Neon | Non-commercial / free tiers |

## System Boundaries

Request path: Browser → Vercel (Next.js) → `/api/*` forwarded with real client IP header + shared secret → NGINX → Spring Boot API → Neon, Redis, Stripe, Cloudinary, SMTP. Stripe webhooks go Stripe → NGINX → API.

Backend, one package per feature area (controller → service → repository):
- `auth` — sign-in, refresh, invitations, password reset, guest tokens
- `branches` — branches, buildings, branch settings, tax rules
- `rooms` — rooms, room types, themes, amenities, blocks, cleaning
- `availability` — nightly room-type inventory counters, search, price quote
- `reservations` — checkout intents, reservations, changes, no-show, cancellation
- `payments` — Stripe, webhooks, staff-recorded payments, refunds, disputes
- `billing` — folio, charges, invoices, outstanding balances
- `notifications` — email outbox, in-app alerts
- `reports` — dashboards, PDF reports
- `audit` — append-only audit log
- `admin` — global settings, master lists, legal versions, data requests

Frontend (Next.js App Router) route groups: public guest pages, `/staff/*`, `/admin`.

## Storage Model

- **PostgreSQL (Neon)**: all business data, checkout intents (deleted after completion or 24 h), audit log, outbox. App uses the pooled connection; Flyway uses the direct connection
- **Cloudinary**: room photos (public). Bank transfer receipts are private, referenced by public id only, opened with short-lived signed links
- **Redis**: shared cache, rate-limit counters, short-lived data. Checkout must not depend on it
- **Loki (via Logback JSON + Alloy)**: sign-in events, kept 30 days. The database keeps only `last_login_at`
- **Stripe**: card data. It never reaches our servers

Schema lives in `hotel_schema_v4.1.md` (39 tables), written to follow v0.3 plus the v4.1 review fixes. The project documentation v0.4 matches it. If they ever differ on tables or columns, the schema file wins. Do not change it until Monty says go.

## Auth and Access Model

- Staff sign in with email + password. 15-minute JWT access token, 7-day rotating refresh token (stored hashed, reuse ends the session family), both in httpOnly Secure cookies
- 5 failed sign-ins in 15 minutes lock the account for 30 minutes; a password reset does not clear it. Sign-in is also limited to 5 attempts per 15 minutes per IP + email
- Staff accounts are created by invitation: one-time link, 48 h, token used up when the password is submitted (not on page load). Password reset link: 10 minutes. Password 8 to 64 characters
- Branch Manager invites only Receptionists and Supervisors of their own branch. Only a Super Admin invites Managers or Super Admins. First Super Admin comes from a seed script
- Guests have no accounts: a private link with a long random token (hash stored, tied to one reservation, valid until 30 days after checkout). History uses a separate token and a short history session
- Role and branch are checked on the server for every request. `proxy.ts` is a convenience only. Receptionist, Supervisor and Manager see only their own branch; Super Admin sees all
- CSRF token/header on every state-changing request; refresh cookie limited to the refresh path; CORS limited to known origins

## Invariants

1. A room type is never sold more than it has rooms: each night is claimed with a guarded update on `room_type_inventory`, in date order
2. Two reservations cannot hold the same room on overlapping dates (exclusion constraint on Pending and Approved)
3. Reservations and payments are never deleted, only cancelled or voided. The audit log is append-only
4. The server calculates all prices, taxes, totals and refunds. Never trust the browser
5. Raw card numbers, expiry or CVV never appear in any form, log or column
6. A card reservation is created only after the payment succeeds. Booking details are saved in a checkout intent first. Pay-after-checkout creates the reservation only after the email code is accepted
7. Stripe webhooks: check signature, process each event once, idempotency keys on payment calls. If the room is gone or the intent is missing, refund automatically
8. Money is whole minor units + currency code. Dates are branch-timezone dates. Timestamps are UTC. All cutoff logic uses the branch timezone
9. Taxes and prices are copied onto the reservation so old bookings never change
10. Tokens (invitation, reset, manage, history) are stored hashed, single-purpose, and never appear in logs, Sentry or PostHog (`/manage`, `/history` paths)
11. Database changes only through Flyway migrations, forward only
12. Every automatic refund needs no approver; goodwill refunds follow the Supervisor limit, then Manager / Super Admin. Nobody decides their own approval request
13. Every reservation starts as Pending. Statuses are Pending, Approved, Cancelled, No Show. Cancelled and No Show are final
