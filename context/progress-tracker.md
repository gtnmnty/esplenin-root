# Progress Tracker

Update this file after every meaningful implementation change.

## Current Phase

- Not started. Planning is complete (docs v0.4, schema v4.1). Implementation begins with Phase 0 Foundation

## Current Goal

- Start Phase 0 (Foundation): repositories, CI, Docker Compose, staging, migrations (a new `V1__init.sql` from schema v4.1), sign-in, roles and branch rules

## Completed

- Project Documentation v0.4 (2026-11-24): brought in line with schema v4.1. v0.3 (2026-11-15) closed all open items
- Database schema v4.1 (39 tables) written as `hotel_schema_v4.1.md` (2026-11-24). Not yet run against Postgres. The Neon `public` schema was reset on both branches; the old Flyway V1 and V2 files are still to be replaced by the new V1
- Context files drafted from v0.3: project-overview, architecture, code-standards, ui-context, ai-workflow-rules

## In Progress

- None yet.

## Next Up

- Phase 0 Foundation (S14). Then Phase 1 Guest booking: branches, rooms, availability, price quote, booking form, checkout intent, card payment, webhook, confirmation, manage booking, cancellation, automatic refunds

## Open Questions

- Write `V1__init.sql` from `hotel_schema_v4.1.md` and run it on a throwaway Postgres 18 before it goes near Neon
- UI: tokens, fonts (all free, OFL) and scaling are set in `ui-context.md`. Still to do: verify contrast of the status chips and `--text-muted` with a checker, and confirm French accent coverage of the fonts
- Exact folder layout inside both repos is not defined
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
