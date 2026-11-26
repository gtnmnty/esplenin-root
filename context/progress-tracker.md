# Progress Tracker

Update this file after every meaningful implementation change.

## Current Phase

- Phase 0 Foundation under way. Planning is complete (docs v0.4, schema v4.1)

## Current Goal

- Finish Phase 0 (Foundation): staging, apply V1 to Neon, then sign-in, roles and branch rules. Done so far: repositories, Docker Compose, V1 migration, CI and baseline (Part 1)

## Completed

- Project Documentation v0.4 (2026-11-24): brought in line with schema v4.1. v0.3 (2026-11-15) closed all open items
- Database schema v4.1 (39 tables) written as `hotel_schema_v4.1.md` (2026-11-24). `V1__init.sql` now holds the whole schema and was applied cleanly to a throwaway Postgres 18 (2026-11-25). The Neon `public` schema was reset on both branches; the new V1 is not applied to Neon yet
- Context files drafted from v0.3: project-overview, architecture, code-standards, ui-context, ai-workflow-rules
- Phase 0 Part 1, CI and baseline (2026-11-26), complete except branch protection (see below):
  - Merged: backend and frontend CI, Dependabot and `.env.example`; Postgres 18 volume fix; local Postgres on host port 5433
  - Backend CI (`verify` and `scan`) also runs on pushes to `main`, with cached scan data (PR #7). Owner confirmed the run on `main` passed. The first backend CI PR (#3) was merged while failing; PR #6 fixed it (dummy datasource settings for tests, correct NVD key option)
  - Frontend: owner confirmed the CI and Dependabot runs in Actions are green
  - The scan suppresses CVE-2022-31691 for `spring-boot-devtools` only: Spring's advisory names IDE YAML editors, not this library
  - Dependabot ignores Spring Boot minor and major updates (pinned to 4.1.x)
  - Branch protection is not enforced: GitHub reports the frontend ruleset needs a GitHub Team organization account on a private repo. Blocking a failing PR is untested. Until then, check that CI is green before every merge
  - Dependency updates squash merged: backend Spring Boot 4.1.3 and Maven 3.10.2 (owner confirmed both CI jobs passed on `main` at `c753d7f`); frontend Tailwind CSS and `@tailwindcss/turbopack` 4.3.5 (owner's local `pnpm build` passed afterwards)
  - Held back: React 19.3.1 (the PR does not update `react-dom`; update both together), TypeScript 6 (compatibility not checked), `@types/node` 26 (do not merge, Node is pinned to 24)

## In Progress

- Phase 0: `V1__init.sql` written from schema v4.1 and applied cleanly to a throwaway Postgres 18 by `./mvnw clean verify` (2026-11-25). Spring Boot Admin removed from `pom.xml` (it broke context startup). Backend layout decided: package by feature. Backend `chore/dev-environment` (remove Spring Boot Admin, pin Mailpit, pin Postgres 18 and Redis 8.2 test containers) was merged as PR #2. Root repo now has a GitHub remote. Env files in both repos have the shared values inlined (the `.env.common` files can be deleted)

## Next Up

- Phase 0 Part 2, backend base: shared code (error handler, request ID, injected `Clock`, base entities, typed settings), deny-by-default security with headers and CORS, and apply V1 to Neon
- Rest of Phase 0 Foundation (S14): Part 3 sign-in and access, Part 4 first Super Admin and passwords, Part 5 staging. Then Phase 1 Guest booking: branches, rooms, availability, price quote, booking form, checkout intent, card payment, webhook, confirmation, manage booking, cancellation, automatic refunds

## Open Questions

- UI: tokens, fonts (all free, OFL) and scaling are set in `ui-context.md`. Still to do: verify contrast of the status chips and `--text-muted` with a checker, and confirm French accent coverage of the fonts
- Dependencies from `architecture.md` that are not in `pom.xml` yet (jjwt, MapStruct, springdoc, Stripe, Cloudinary, Bucket4j, Resilience4j, Caffeine, ShedLock, Sentry): add each with the feature that first needs it
- Oracle Always Free may reclaim an idle server. Check CPU, network and memory after one week

## Architecture Decisions

- Guests have no accounts; staff only. Guests use private token links (keeps booking simple, avoids password handling)
- Cancel only, never delete (keeps a complete history for audit and refunds)
- Room types are per branch; theme is part of the room type
- Reservation always starts Pending; a receptionist assigns the room (separates payment from room assignment)
- Card booking: save a checkout intent first, create the reservation after the payment webhook (a successful payment can always become a reservation; abandoned payments leave nothing)
- Availability tracked per room type per night with guarded counter updates, plus an exclusion constraint per room (prevents double booking under load)
- No cancellation, late-cancellation or no-show fees; all refunds are automatic (simpler rules, no approvers)
- No-show cutoff is the end of the check-in date in branch timezone; Pending reservations expire at the same moment
- Date changes: guests cannot change; a Receptionist can change twice, then a Supervisor must allow it with a reason
- Check-in needs the "ID verified?" button first (records who checked the ID and the document type)
- Pay-after-checkout email code: at most 3 code requests per intent in 15 minutes and 5 wrong guesses per code, counted in the database so checkout never depends on Redis
- A goodwill refund or room-type change request is never decided by the employee who made it
- Frontend forwards `/api/*` so cookies stay first-party; NGINX trusts the real-client-IP header only with a shared secret
- Sign-in events go to Loki only; the database keeps `last_login_at`
- Self-hosted GitHub runner on the Oracle server; staging exists on the same server
- `proxy.ts` replaces `middleware.ts`; PDF only for reports
- Phase 0 decisions (2026-11-26):
  - Sign-in lock window is counted from the first failure. Needs a first-failure timestamp column on `employees`, added by a V2 migration in Part 3 (owner approved this schema change; update the schema file with it)
  - Staging is set up in Phase 0 after Parts 1 to 4, with NGINX limited to TLS, routing and trusted client IP
  - Sign-in rate-limit counts use Bucket4j in memory behind an interface, moving to Redis later
  - Forgot password: 3 requests per 15 minutes per email, and 5 per 15 minutes per IP. Known and unknown emails get the same response. Add the numbers to S08
  - Sign-in shows Pending or suspended status only after a correct password. A Pending account has no password, so it gets the generic error
  - Frontend layout: routes in `app/`, feature code in `features/<name>/`
  - CI uses GitHub-hosted runners until the Oracle self-hosted runner exists (temporary departure from S12)

## Session Notes

- The documentation file is the source of truth. These context files summarize it
- Build order: core booking flow first, monitoring stack last
- Solo project, one-month timeline
