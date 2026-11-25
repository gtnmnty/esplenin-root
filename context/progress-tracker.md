# Progress Tracker

Update this file after every meaningful implementation change.

## Current Phase

- Phase 0 Foundation under way. Planning is complete (docs v0.4, schema v4.1)

## Current Goal

- Finish Phase 0 (Foundation): CI, staging, apply V1 to Neon, then sign-in, roles and branch rules. Done so far: repositories, Docker Compose, V1 migration

## Completed

- Project Documentation v0.4 (2026-11-24): brought in line with schema v4.1. v0.3 (2026-11-15) closed all open items
- Database schema v4.1 (39 tables) written as `hotel_schema_v4.1.md` (2026-11-24). `V1__init.sql` now holds the whole schema and was applied cleanly to a throwaway Postgres 18 (2026-11-25). The Neon `public` schema was reset on both branches; the new V1 is not applied to Neon yet
- Context files drafted from v0.3: project-overview, architecture, code-standards, ui-context, ai-workflow-rules

## In Progress

- Phase 0: `V1__init.sql` written from schema v4.1 and applied cleanly to a throwaway Postgres 18 by `./mvnw clean verify` (2026-11-25). Spring Boot Admin removed from `pom.xml` (it broke context startup). Backend layout decided: package by feature. Backend `chore/dev-environment` has 3 commits (remove Spring Boot Admin, pin Mailpit, pin Postgres 18 and Redis 8.2 test containers) waiting to be merged. Root repo now has a GitHub remote. Env files in both repos have the shared values inlined (the `.env.common` files can be deleted)

## Next Up

- Phase 0 Foundation (S14). Then Phase 1 Guest booking: branches, rooms, availability, price quote, booking form, checkout intent, card payment, webhook, confirmation, manage booking, cancellation, automatic refunds

## Open Questions

- UI: tokens, fonts (all free, OFL) and scaling are set in `ui-context.md`. Still to do: verify contrast of the status chips and `--text-muted` with a checker, and confirm French accent coverage of the fonts
- Backend folder layout is decided (by feature). Frontend layout is still not defined
- Dependencies from `architecture.md` that are not in `pom.xml` yet (jjwt, MapStruct, springdoc, Stripe, Cloudinary, Bucket4j, Resilience4j, Caffeine, ShedLock, Sentry): add each with the feature that first needs it
- Frontend: `@types/node` is `^20` but the docs say Node 24. Align when frontend work starts
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

## Session Notes

- The documentation file is the source of truth. These context files summarize it
- Build order: core booking flow first, monitoring stack last
- Solo project, one-month timeline
