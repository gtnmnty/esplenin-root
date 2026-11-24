# Code Standards

Source: Esplenin Hotel Project Documentation v0.4 (S05 to S13). Where this file is silent, follow the existing code and ask.

## General

- Keep modules small and single-purpose. One backend package per feature area
- Fix root causes, do not layer workarounds
- Do not mix unrelated concerns in one class, component or route
- Every number (times, limits, percentages) comes from the S08 Default Values list or branch settings. No hardcoded copies in code
- Texts shown to users are ready for translation (English, French, Japanese, Chinese)

## Java / Spring Boot (Java 21, Spring Boot 4.1.x)

- Layering: controller → service → repository. Controllers stay thin; business rules live in services
- Map entities to DTOs with MapStruct. Never return JPA entities from an API
- Validate input at the edge with Hibernate Validator. Never trust the client for prices, taxes, totals or refund amounts
- Database access only through JPA with parameters. No string-built SQL
- All errors go through one exception handler and use RFC 7807 problem details with a stable error code
- Enforce role and branch with method-level security on every endpoint. Check ownership before any mutation
- Use optimistic locking (`version`) on records people edit at the same time
- Use a controllable clock (injected `Clock`) for time logic so tests can move time
- Use Lombok sparingly (boilerplate only)

## TypeScript / Next.js (Next.js 16.x, Node 24)

- Strict mode on. Avoid `any`; use explicit types
- Validate forms with React Hook Form + Zod. Validate unknown external data before trusting it
- Server components by default. Add `"use client"` only when browser interactivity needs it
- Data fetching with TanStack Query. All backend calls go through `/api/*` (forwarded by Next.js)
- Route guards live in `proxy.ts` (not `middleware.ts`). They are a convenience; the API enforces access
- Translate every user-facing string with next-intl. Currency comes from the branch, language from the visitor
- Do not load PostHog on `/manage` or `/history`, and only after consent

## Styling

- Tailwind with CSS custom property tokens. No hardcoded hex values or raw color classes (see `ui-context.md`)
- Reuse the shared UI pieces: search bar, room type card, price breakdown card, status chip, KPI card, chart card, data table, side panel and modal, empty/loading/error states
- Guest pages fully responsive; staff pages desktop and tablet first
- WCAG 2.1 AA: keyboard use, contrast, labels, focus order, screen reader text

## API Routes

- Base path `/api/v1`, JSON only, documented in Swagger UI
- Validate and parse input before any logic runs
- Enforce auth and ownership before any mutation
- Dates are ISO dates in the branch timezone. Timestamps are ISO in UTC. Money is an integer in minor units plus a currency code
- Lists use `page` and `size` with a maximum size and a stable sort
- Payment and booking creation accept an idempotency key
- Every response carries a request ID that also appears in the logs
- Apply rate limits per endpoint group; stricter on sign-in, booking, email-code and email-link requests

## Data and Storage

- Business data in PostgreSQL. Schema changes only through Flyway migrations (forward only; fix mistakes with a new migration)
- Reservations and payments are never deleted, only cancelled or voided
- Audit log is append-only. Write an entry for every important action (including reasons for overrides)
- Keys are time-ordered UUIDs (`checkout_intents` uses a random one so it cannot be guessed). Statuses and types are text with a CHECK constraint, not database enums
- Receipts: JPG, PNG, PDF only, 5 MB max, real file type checked, private Cloudinary files, store only the public id
- Copy taxes and prices onto the reservation at booking time

## Security and Logging

- Secrets only in environment variables or GitHub Secrets. Never in the repository
- No password, token, full card data or typed login email in logs. Mask emails where possible
- Never put raw card data in any form, log or column
- Security headers: CSP, HSTS, no-sniff, `Referrer-Policy: no-referrer`
- Dependency scanning in CI: Dependabot, `npm audit`, OWASP Dependency-Check

## Testing (Definition of Done per feature)

- Matches the S02 scope numbers it covers
- Tests written and passing in CI (JUnit 5 + Testcontainers, Vitest, Playwright). Coverage target: 80% backend overall, 90% on booking, payment, no-show and refund
- Access rules checked for every role (role-matrix test)
- Errors and empty states handled; texts ready for translation
- Audit log entries written for important actions
- This documentation and the schema file updated
- Self-reviewed against this checklist

## Git

- Two repos (backend, frontend). `main` is protected and always deployable: pull request and passing CI required
- Branch names: `feature/auth-login`, `fix/...`, `docs/...`, `chore/...` (lowercase, hyphens). One feature per branch. Same branch name in both repos for cross-repo features; merge backend first
- Squash merge, then delete the branch. Tag releases (`v0.1.0`)
- Commit format: `type(scope): short summary`, types `feat, fix, docs, refactor, test, chore, ci`. Imperative, 72 characters or less. Do not use "update" or "implement"
- Every PR lists the S02 scope item numbers it covers, how it was tested, and screenshots for UI changes
- If two branches use the same Flyway number, the second renames its file after rebasing

## File Organization

- Backend repo: one package per module: `auth`, `branches`, `rooms`, `availability`, `reservations`, `payments`, `billing`, `notifications`, `reports`, `audit`, `admin`
- Frontend repo: Next.js App Router with route groups for public guest pages, `/staff/*` and `/admin`
- `components/ui/` — generated shadcn/ui components (add with the CLI, do not hand-edit)
- Exact folder names beyond these are not defined yet. Confirm before creating new top-level folders
