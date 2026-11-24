# Test Cases

Project: Esplenin Hotel Reservation and Payment System (docs v0.4)

Update the Status column after every test run. The test suite and CI are the source of truth for results. This file is the checklist.

## How to Use

- Every case has an ID: `TC-[scope number]-[two digits]`, for example `TC-014-01`. The scope number is from S02 "Each role can do"
- Cases from the S02 "Each role cannot do" list use `TC-X[cannot number]-01`, for example `TC-X25-01`
- Put the ID at the start of the real test name, so a case can be matched to its test
- `/test` updates Status after each run
- A `Must` case that is `FAILING` or `NOT WRITTEN` blocks a release

## Status Key

| Status | Meaning |
| --- | --- |
| `NOT WRITTEN` | Case is listed, test does not exist yet |
| `PASSING` | Automated test exists and passed on the last run |
| `FAILING` | Automated test exists and failed on the last run |
| `MANUAL-PENDING` | Must be checked by hand, not done yet |
| `MANUAL-PASSED` | Checked by hand and worked |
| `SKIPPED` | Intentionally not run, reason in Notes |

## Priority Key

| Priority | Meaning |
| --- | --- |
| `Must` | Release is blocked if it fails or is missing |
| `Should` | Fix soon, does not block |
| `Could` | Nice to have |

## Type Key

`Unit` (JUnit 5, Mockito, Vitest), `Integration` (Testcontainers with real PostgreSQL, Flyway), `E2E` (Playwright), `Concurrency` (parallel requests), `Manual`

## Defaults Used In These Cases (from S08)

Max stay 14 nights. Max advance booking 5 days. Down payment at least 30%. Receptionist date changes 2 per reservation. Email code 6 digits, 10 minutes, 5 wrong guesses per code, 3 code requests per 15 minutes. 2 active unpaid reservations per email. Sign-in lock after 5 failures in 15 minutes for 30 minutes. Invitation link 48 hours. Reset link 10 minutes. Manage link until 30 days after checkout. If a number changes in S08, change it here too.

## Guest: Search and Price

Scope items: 1, 2, 3, 4, 5, 17

| ID | Scope | Case | Type | Priority | Status | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| TC-001-01 | 1 | Room types page shows photos, amenities, capacity and price per night | Integration | Should | NOT WRITTEN |  |
| TC-002-01 | 2 | Branch dropdown lists active branches only; a closed branch is not selectable | Integration | Should | NOT WRITTEN |  |
| TC-003-01 | 3 | Search returns only room types that are free on every night for the dates and guest count | Integration | Must | NOT WRITTEN |  |
| TC-003-02 | 3 | Check-out on or before check-in is rejected | Unit | Must | NOT WRITTEN |  |
| TC-003-03 | 3 | A stay of 14 nights is accepted and 15 nights is refused | Unit | Must | NOT WRITTEN | Max stay default |
| TC-003-04 | 3 | Check-in 5 days ahead is accepted and 6 days ahead is refused | Unit | Must | NOT WRITTEN | Uses branch timezone and controllable clock |
| TC-003-05 | 3 | Guest count above the room type capacity is refused | Unit | Should | NOT WRITTEN |  |
| TC-004-01 | 4 | Theme is Standard by default; choosing another theme filters the results | Integration | Should | NOT WRITTEN |  |
| TC-005-01 | 5 | Price breakdown equals nightly rate x nights plus taxes, matching the server calculation exactly | Unit | Must | NOT WRITTEN | Minor units |
| TC-005-02 | 5 | Percentage and fixed taxes, included and added, give the correct total | Unit | Must | NOT WRITTEN |  |
| TC-005-03 | 5 | A price or total sent by the browser is ignored; the server recalculates | Integration | Must | NOT WRITTEN |  |
| TC-005-04 | 5 | 'Today' and all cutoffs use the branch timezone, not the server timezone | Unit | Must | NOT WRITTEN |  |
| TC-017-01 | 17 | Language switch changes all visible text in English, French, Japanese and Chinese | E2E | Should | NOT WRITTEN |  |
| TC-017-02 | 17 | Prices show in the branch currency with no conversion | Integration | Must | NOT WRITTEN |  |

## Guest: Booking and Payment

Scope items: 6, 7, 8, 9, 15, 22

| ID | Scope | Case | Type | Priority | Status | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| TC-006-01 | 6 | Valid main guest details (name, email, phone, country) create a checkout intent | Integration | Must | NOT WRITTEN |  |
| TC-006-02 | 6 | Missing or invalid name, email, phone or country is rejected with a clear error | Unit | Must | NOT WRITTEN |  |
| TC-006-03 | 6 | /book, code request and history-link request return 429 when the rate limit is exceeded | Integration | Should | NOT WRITTEN |  |
| TC-007-01 | 7 | Booking is refused unless terms, privacy policy and cancellation policy are accepted | Integration | Must | NOT WRITTEN |  |
| TC-007-02 | 7 | The accepted version of each document is stored with the reservation | Integration | Must | NOT WRITTEN |  |
| TC-008-01 | 8 | Successful card payment creates a Pending reservation after the webhook arrives | Integration | Must | NOT WRITTEN | Stripe test mode |
| TC-008-02 | 8 | Failed or abandoned payment leaves no reservation | Integration | Must | NOT WRITTEN |  |
| TC-008-03 | 8 | Checkout intents are deleted after completion or after 24 hours | Integration | Should | NOT WRITTEN |  |
| TC-008-04 | 8 | Down payment of at least 30% of the grand total incl. taxes is accepted; below 30% is refused | Unit | Must | NOT WRITTEN |  |
| TC-008-05 | 8 | Down payment is refused when the branch does not offer it | Integration | Must | NOT WRITTEN |  |
| TC-008-06 | 8 | Down payment amount stays frozen; later changes adjust the balance, not the down payment | Integration | Should | NOT WRITTEN |  |
| TC-008-07 | 8 | No raw card number, expiry or CVV appears in any request, log or database column | Integration | Must | NOT WRITTEN | Scan logs and schema |
| TC-008-08 | 8 | The same booking or payment request with the same idempotency key creates one reservation | Integration | Must | NOT WRITTEN |  |
| TC-009-01 | 9 | Pay after checkout is offered only when the branch allows it and the email has no outstanding balance | Integration | Must | NOT WRITTEN |  |
| TC-009-02 | 9 | A correct 6-digit email code creates a Pending reservation | Integration | Must | NOT WRITTEN |  |
| TC-009-03 | 9 | The code expires after 10 minutes | Unit | Must | NOT WRITTEN | Controllable clock |
| TC-009-04 | 9 | A wrong code can be retried; the correct code still works within 10 minutes while fewer than 5 wrong guesses were made | Integration | Must | NOT WRITTEN | Per S08: 5 wrong guesses per code |
| TC-009-05 | 9 | Code verification is rate limited per email and IP | Integration | Must | NOT WRITTEN | In addition to the per-code and per-intent limits (TC-009-07, TC-009-08) |
| TC-009-06 | 9 | A third active unpaid reservation by the same email is refused; staff bookings are exempt | Integration | Must | NOT WRITTEN |  |
| TC-009-07 | 9 | The 5th wrong guess revokes the code; the correct code is refused afterwards and a new code is needed | Integration | Must | NOT WRITTEN | Counter is in the database, not Redis |
| TC-009-08 | 9 | A fourth code request for the same checkout intent within 15 minutes is refused; a new code revokes the old one | Integration | Must | NOT WRITTEN | Controllable clock |
| TC-015-01 | 15 | Guest pays the remaining balance online; payment status becomes Paid | Integration | Must | NOT WRITTEN |  |
| TC-015-02 | 15 | A down payment guest pays only the difference after a staff change | Integration | Should | NOT WRITTEN |  |
| TC-022-01 | 22 | A guest with an outstanding balance cannot start a new booking at any branch | Integration | Must | NOT WRITTEN |  |

## Guest: Manage, Cancel and History

Scope items: 10, 11, 12, 13, 14, 16, 18, 19, 20

| ID | Scope | Case | Type | Priority | Status | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| TC-010-01 | 10 | Reservation page shows the correct status: Pending, Approved, Cancelled or No Show | E2E | Must | NOT WRITTEN |  |
| TC-011-01 | 11 | Confirmation email is sent with the private manage link | Integration | Must | NOT WRITTEN | Mailpit in dev |
| TC-011-02 | 11 | Approval, cancellation, refund and charge-added emails are sent on the matching event | Integration | Should | NOT WRITTEN |  |
| TC-011-03 | 11 | Check-in reminder is queued around 09:00 branch time for tomorrow's arrivals; review request is sent after checkout | Integration | Should | NOT WRITTEN | Controllable clock |
| TC-011-04 | 11 | When SMTP is down, emails wait in the outbox and retry; booking still works | Integration | Should | NOT WRITTEN |  |
| TC-011-05 | 11 | Emails are sent in the guest's language | Integration | Should | NOT WRITTEN |  |
| TC-012-01 | 12 | A valid manage token shows the booking; a wrong token or another reservation's token is refused | Integration | Must | NOT WRITTEN |  |
| TC-012-02 | 12 | The manage link works until 30 days after checkout and is refused after | Unit | Must | NOT WRITTEN | Controllable clock |
| TC-012-03 | 12 | The confirmation page shows no data when opened by reservation number | Integration | Must | NOT WRITTEN |  |
| TC-012-04 | 12 | Tokens are stored hashed and never appear in logs, Sentry or PostHog on /manage and /history | Integration | Must | NOT WRITTEN |  |
| TC-013-01 | 13 | History-link request emails a link only to that address; link is valid 30 days | Integration | Must | NOT WRITTEN |  |
| TC-013-02 | 13 | History page lists only bookings under that email | Integration | Must | NOT WRITTEN |  |
| TC-014-01 | 14 | Guest cancels a Pending reservation: full automatic refund, no fee | Integration | Must | NOT WRITTEN |  |
| TC-014-02 | 14 | Guest cancels an Approved reservation: room and nights are released | Integration | Must | NOT WRITTEN |  |
| TC-014-03 | 14 | Cancelling on the check-in date before check-in is allowed with a full refund | Unit | Must | NOT WRITTEN | No free-cancellation window |
| TC-014-04 | 14 | Stripe payments refund to the card, newest payment first; cash, transfer and external card get a Pending refund | Integration | Must | NOT WRITTEN |  |
| TC-016-01 | 16 | Invoice and receipt PDF show correct amounts, taxes and currency | Integration | Should | NOT WRITTEN |  |
| TC-018-01 | 18 | A review can only be left after checkout | Integration | Could | NOT WRITTEN |  |
| TC-019-01 | 19 | Marketing consent is stored separately and can be withdrawn | Integration | Should | NOT WRITTEN |  |
| TC-020-01 | 20 | Data export request returns the guest's data as a file | Integration | Could | NOT WRITTEN |  |
| TC-020-02 | 20 | Cancel after check-in, after checkout, or on a Cancelled or No Show booking is refused | Integration | Must | NOT WRITTEN |  |

## Staff: Sign In and Accounts

Scope items: 21, 46, 57

| ID | Scope | Case | Type | Priority | Status | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| TC-021-01 | 21 | Valid sign-in sets httpOnly Secure cookies; access token lasts 15 minutes, refresh token 7 days | Integration | Must | NOT WRITTEN |  |
| TC-021-02 | 21 | 5 failed sign-ins in 15 minutes lock the account for 30 minutes | Integration | Must | NOT WRITTEN | Controllable clock |
| TC-021-03 | 21 | A password reset does not clear the lock | Integration | Must | NOT WRITTEN |  |
| TC-021-04 | 21 | The 6th sign-in attempt in 15 minutes for the same IP and email returns 429 | Integration | Must | NOT WRITTEN |  |
| TC-021-05 | 21 | Refresh token rotates; reusing an old refresh token ends the session family | Integration | Must | NOT WRITTEN |  |
| TC-021-06 | 21 | Login logs contain no password and no typed email | Integration | Must | NOT WRITTEN |  |
| TC-021-07 | 21 | A state-changing request without the CSRF token is refused | Integration | Must | NOT WRITTEN |  |
| TC-021-08 | 21 | Password of 8 to 64 characters is accepted; 7 and 65 are refused | Unit | Must | NOT WRITTEN |  |
| TC-021-09 | 21 | Password reset link expires after 10 minutes | Unit | Must | NOT WRITTEN | Controllable clock |
| TC-046-01 | 46 | Manager invites by email; account stays Pending until the password is set | Integration | Must | NOT WRITTEN |  |
| TC-046-02 | 46 | Opening the invitation link does not use it up; submitting the password does | Integration | Must | NOT WRITTEN |  |
| TC-046-03 | 46 | Expired (48 hours), used and cancelled invitation links are refused | Integration | Must | NOT WRITTEN |  |
| TC-046-04 | 46 | Resending an invitation makes the old link stop working | Integration | Should | NOT WRITTEN |  |
| TC-046-05 | 46 | Suspending or deactivating an account blocks sign-in and ends its sessions | Integration | Must | NOT WRITTEN |  |
| TC-046-06 | 46 | Manager can invite only Receptionists and Supervisors of their own branch | Integration | Must | NOT WRITTEN |  |
| TC-046-07 | 46 | Role changes by a Manager go up to Supervisor only | Integration | Must | NOT WRITTEN |  |
| TC-057-01 | 57 | Super Admin manages employees of every branch, including Managers and Super Admins | Integration | Must | NOT WRITTEN |  |
| TC-057-02 | 57 | First Super Admin is created by the seed script | Manual | Should | MANUAL-PENDING |  |

## Staff: Reservations and Changes

Scope items: 22, 23, 24, 25, 26, 27, 34

| ID | Scope | Case | Type | Priority | Status | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| TC-022-02 | 22 | Walk-in and phone reservations are created straight away as Pending | Integration | Must | NOT WRITTEN |  |
| TC-022-03 | 22 | Staff bookings are exempt from the 2 unpaid reservations per email limit | Integration | Must | NOT WRITTEN |  |
| TC-023-01 | 23 | Search by reservation number, guest name, email or date returns only own-branch results | Integration | Must | NOT WRITTEN |  |
| TC-024-01 | 24 | Dates and guest count can be changed while Pending or Approved; refused when Cancelled or No Show | Integration | Must | NOT WRITTEN |  |
| TC-024-02 | 24 | Extension is refused when an added night is full or the assigned room has another reservation | Integration | Must | NOT WRITTEN |  |
| TC-024-03 | 24 | Early checkout recalculates nights stayed (minimum 1) and refunds an overpayment automatically | Integration | Must | NOT WRITTEN |  |
| TC-024-04 | 24 | Shortening a stay recalculates the total and refunds the difference | Integration | Must | NOT WRITTEN |  |
| TC-024-05 | 24 | An underpayment after a change becomes a balance; a down payment guest pays only the difference | Integration | Must | NOT WRITTEN |  |
| TC-024-06 | 24 | A date change that makes the assigned room unavailable releases the room and returns the booking to Pending | Integration | Must | NOT WRITTEN |  |
| TC-025-01 | 25 | Receptionist's 1st and 2nd date change succeed; the 3rd is refused | Integration | Must | NOT WRITTEN |  |
| TC-025-02 | 25 | Supervisor allows a 3rd change with a reason; without a reason it is refused | Integration | Must | NOT WRITTEN |  |
| TC-025-03 | 25 | Every date change is logged, recalculates the price and resets the no-show timer | Integration | Must | NOT WRITTEN |  |
| TC-025-04 | 25 | Reschedule with an empty check-out keeps the same number of nights | Unit | Must | NOT WRITTEN |  |
| TC-026-01 | 26 | Staff cancellation refunds everything automatically; no endpoint deletes a reservation | Integration | Must | NOT WRITTEN |  |
| TC-026-02 | 26 | A Pending reservation whose check-in date has ended is cancelled (reason: Pending expired), nights released, payments refunded | Integration | Must | NOT WRITTEN | Job runs every 15 minutes |
| TC-027-01 | 27 | Assigning a matching room sets the room Reserved and the reservation Approved | Integration | Must | NOT WRITTEN |  |
| TC-027-02 | 27 | Assigning a room with overlapping dates is refused | Integration | Must | NOT WRITTEN |  |
| TC-027-03 | 27 | A room of a different room type is refused | Integration | Must | NOT WRITTEN |  |
| TC-027-04 | 27 | A room can be assigned before any payment is made | Integration | Should | NOT WRITTEN |  |
| TC-034-01 | 34 | Manual No Show is allowed on or after the check-in date and refused before it | Integration | Must | NOT WRITTEN |  |
| TC-034-02 | 34 | Manual No Show refunds everything automatically | Integration | Must | NOT WRITTEN |  |

## Staff: Check-in, Check-out and Rooms

Scope items: 29, 30, 31, 35, 36, 37

| ID | Scope | Case | Type | Priority | Status | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| TC-029-01 | 29 | Check-in sets the room Occupied; the ID verified step has already recorded the document type | Integration | Must | NOT WRITTEN |  |
| TC-029-02 | 29 | Check-in is refused while the room is Cleaning | Integration | Must | NOT WRITTEN |  |
| TC-029-03 | 29 | ID document type list matches the agreed list | Unit | Should | NOT WRITTEN | Passport, Driver's license, National ID, Other |
| TC-029-04 | 29 | Check-in is refused until the ID verified step is done; the step records who verified and when | Integration | Must | NOT WRITTEN | Database CHECK as well as the service rule |
| TC-030-01 | 30 | Check-out sets the room Cleaning, then Available after the cleaning time | Integration | Must | NOT WRITTEN | Controllable clock; job runs every minute |
| TC-030-02 | 30 | Cleaning time 0 skips the Cleaning state | Unit | Should | NOT WRITTEN |  |
| TC-030-03 | 30 | Check-out with a balance owed is refused unless the guest chose pay after checkout | Integration | Must | NOT WRITTEN |  |
| TC-030-04 | 30 | A pay-after-checkout guest who leaves owing money gets an outstanding balance under their email | Integration | Must | NOT WRITTEN |  |
| TC-031-01 | 31 | Transfer to another room of the same room type succeeds | Integration | Must | NOT WRITTEN |  |
| TC-035-01 | 35 | Room status precedence is Unavailable, Occupied, Cleaning, Reserved, Available | Unit | Must | NOT WRITTEN |  |
| TC-035-02 | 35 | Room board shows the right status for past, today and future dates | Integration | Should | NOT WRITTEN |  |
| TC-036-01 | 36 | Arrivals, departures and in-house lists are correct for today in branch time | Integration | Should | NOT WRITTEN |  |
| TC-037-01 | 37 | Notifications are visible only to staff of that branch | Integration | Should | NOT WRITTEN |  |

## Staff: Payments, Charges and Invoices

Scope items: 28, 32, 33

| ID | Scope | Case | Type | Priority | Status | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| TC-028-01 | 28 | Cash payment is recorded and shows settled at once | Integration | Must | NOT WRITTEN |  |
| TC-028-02 | 28 | Bank transfer requires a receipt photo and a reference | Integration | Must | NOT WRITTEN |  |
| TC-028-03 | 28 | External card payment requires a reference; card data is not accepted | Integration | Must | NOT WRITTEN |  |
| TC-028-04 | 28 | Receipts accept JPG, PNG and PDF up to 5 MB; other types and renamed files are rejected | Integration | Must | NOT WRITTEN | Real file type checked |
| TC-028-05 | 28 | Receipt is private; database holds only the public id; link is short-lived | Integration | Should | NOT WRITTEN | Cloudinary |
| TC-028-06 | 28 | Staff can record a Pending refund for cash, transfer or external card payments | Integration | Must | NOT WRITTEN |  |
| TC-032-01 | 32 | Adding a charge updates the bill total and emails the guest | Integration | Must | NOT WRITTEN |  |
| TC-032-02 | 32 | Charges and totals use whole minor units with no rounding drift | Unit | Must | NOT WRITTEN |  |
| TC-033-01 | 33 | Staff can issue an invoice or receipt PDF and resend the confirmation email | Integration | Should | NOT WRITTEN |  |

## Supervisor

Scope items: 38, 39, 40, 41, 42, 43, 44

| ID | Scope | Case | Type | Priority | Status | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| TC-038-01 | 38 | Supervisor can void or correct a charge or invoice; the entry is kept and audited with a reason | Integration | Must | NOT WRITTEN |  |
| TC-039-01 | 39 | Goodwill refund within the branch limit is approved | Integration | Must | NOT WRITTEN |  |
| TC-039-02 | 39 | Goodwill refund above the limit is refused and goes to a Manager | Integration | Must | NOT WRITTEN |  |
| TC-039-03 | 39 | The employee who requested a goodwill refund cannot approve or reject it | Integration | Must | NOT WRITTEN | Database CHECK as well as the service rule |
| TC-040-01 | 40 | Room transfer to a different room type is approved with the price adjusted by the money formula | Integration | Must | NOT WRITTEN |  |
| TC-040-02 | 40 | The employee who requested a room-type change cannot decide it | Integration | Must | NOT WRITTEN |  |
| TC-042-01 | 42 | Blocking a room shows it as Unavailable | Integration | Must | NOT WRITTEN |  |
| TC-042-02 | 42 | Block is refused when an approved reservation holds the room | Integration | Must | NOT WRITTEN |  |
| TC-042-03 | 42 | Block is refused when it would cause overbooking of the room type | Integration | Must | NOT WRITTEN |  |
| TC-042-04 | 42 | Releasing a block makes the room available again | Integration | Should | NOT WRITTEN |  |
| TC-043-01 | 43 | Daily summary shows correct occupancy, arrivals, departures and revenue | Integration | Should | NOT WRITTEN |  |
| TC-044-01 | 44 | Supervisor can read the audit trail of own branch only and cannot change it | Integration | Must | NOT WRITTEN |  |

## Branch Manager

Scope items: 45, 47, 48, 49, 50, 51, 52, 53, 54

| ID | Scope | Case | Type | Priority | Status | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| TC-045-01 | 45 | Manager can approve a refund of any amount | Integration | Must | NOT WRITTEN |  |
| TC-047-01 | 47 | Manager can manage buildings, rooms and room types of own branch only | Integration | Should | NOT WRITTEN |  |
| TC-047-02 | 47 | Changing a room type price does not change existing reservations | Integration | Must | NOT WRITTEN |  |
| TC-047-03 | 47 | Room photo upload works and shows on the guest pages | Manual | Should | MANUAL-PENDING |  |
| TC-048-01 | 48 | Branch settings (max stay, max advance booking, down payment %, pay after checkout, refund limit, cleaning time) are saved and applied | Integration | Must | NOT WRITTEN |  |
| TC-048-02 | 48 | Changing tax rules does not change existing reservations | Integration | Must | NOT WRITTEN |  |
| TC-048-03 | 48 | Down payment percentage below 30% is refused | Unit | Must | NOT WRITTEN |  |
| TC-049-01 | 49 | Manager writes off an own-branch outstanding balance with a reason; booking block is lifted | Integration | Must | NOT WRITTEN |  |
| TC-049-02 | 49 | A Manager of another branch cannot write off the balance | Integration | Must | NOT WRITTEN |  |
| TC-050-01 | 50 | Dashboard figures (revenue, occupancy, ADR, cancellations, no-shows, ratings, outstanding balances) match the data | Integration | Should | NOT WRITTEN |  |
| TC-050-02 | 50 | Dashboard PDF export downloads | Integration | Should | NOT WRITTEN |  |
| TC-051-01 | 51 | Full audit trail of own branch is visible; other branches are not | Integration | Must | NOT WRITTEN |  |
| TC-052-01 | 52 | Disputes exist only for Stripe card payments; the dispute webhook updates the payment | Integration | Should | NOT WRITTEN |  |
| TC-053-01 | 53 | Alerts appear for low availability, unpaid balances left, and refunds waiting to be recorded | Integration | Should | NOT WRITTEN |  |
| TC-054-01 | 54 | Hiding a review needs a reason; hidden reviews are not shown or counted and the action is audited | Integration | Could | NOT WRITTEN |  |

## Super Admin

Scope items: 55, 56, 58, 59, 60, 61, 62

| ID | Scope | Case | Type | Priority | Status | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| TC-055-01 | 55 | Super Admin can create, edit, activate and deactivate branches with country, timezone and currency | Integration | Must | NOT WRITTEN |  |
| TC-055-02 | 55 | Closing a branch cancels its future reservations and refunds everything paid | Integration | Must | NOT WRITTEN |  |
| TC-055-03 | 55 | Closing a branch is refused while guests are checked in | Integration | Must | NOT WRITTEN |  |
| TC-056-01 | 56 | Global defaults apply to new branches | Integration | Should | NOT WRITTEN |  |
| TC-058-01 | 58 | Master lists of themes and amenities and their translations can be managed | Integration | Should | NOT WRITTEN |  |
| TC-059-01 | 59 | New legal versions can be published; new bookings record the new version, old acceptances keep theirs | Integration | Should | NOT WRITTEN |  |
| TC-060-01 | 60 | Anonymization replaces name, email and phone on reservations | Integration | Should | NOT WRITTEN |  |
| TC-060-02 | 60 | Anonymization is rejected while the email has an open outstanding balance | Integration | Should | NOT WRITTEN |  |
| TC-061-01 | 61 | Company dashboards and branch comparisons match the data | Integration | Should | NOT WRITTEN |  |
| TC-062-01 | 62 | Company-wide audit trail is visible to Super Admin only | Integration | Should | NOT WRITTEN |  |

## Critical Scenarios (S13, all must pass)

Scope items: 8, 14, 22, 27, 34, 49

| ID | Scope | Case | Type | Priority | Status | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| TC-008-09 | 8 | Double booking: 50 guests book the last room of a type at once; exactly one succeeds and the others are refunded | Concurrency | Must | NOT WRITTEN | Success criterion; k6 + Testcontainers |
| TC-008-10 | 8 | The same Stripe event delivered twice creates one reservation | Integration | Must | NOT WRITTEN |  |
| TC-008-11 | 8 | Payment succeeds but the room is gone: automatic refund and a message to the guest | Integration | Must | NOT WRITTEN |  |
| TC-008-12 | 8 | Payment succeeds but the checkout intent is missing: automatic refund | Integration | Must | NOT WRITTEN |  |
| TC-008-13 | 8 | Webhook with an invalid signature is rejected | Integration | Must | NOT WRITTEN |  |
| TC-014-05 | 14 | Refund never exceeds the amount paid across all payments | Unit | Must | NOT WRITTEN |  |
| TC-022-04 | 22 | A stayed guest who leaves owing money blocks new bookings by that email at every branch; a no-show does not | Integration | Must | NOT WRITTEN |  |
| TC-027-05 | 27 | Two receptionists assign the same room to overlapping reservations at once; exactly one succeeds | Concurrency | Must | NOT WRITTEN | Exclusion constraint |
| TC-034-03 | 34 | No-show job: an Approved reservation past the cutoff becomes No Show, frees the room and creates a full refund | Integration | Must | NOT WRITTEN | Stripe refunded via Stripe; cash or transfer gets a Pending refund |
| TC-034-04 | 34 | No-show cutoff is 24:00 of the check-in date in the branch timezone: 23:59 is not a no-show, 00:00 is | Unit | Must | NOT WRITTEN |  |
| TC-034-05 | 34 | Unpaid and pay-after-checkout no-shows owe nothing | Integration | Must | NOT WRITTEN |  |
| TC-034-06 | 34 | Each background job runs on one instance only | Integration | Should | NOT WRITTEN | ShedLock |
| TC-049-03 | 49 | Optimistic locking: two staff editing the same reservation at once; the second gets a conflict | Concurrency | Should | NOT WRITTEN |  |
| TC-008-14 | 8 | Stripe down: no new card bookings, but pay after checkout and staff bookings still work | Integration | Should | NOT WRITTEN |  |
| TC-008-15 | 8 | Redis down: caches skipped, rate limits fall back to local, booking still works | Integration | Should | NOT WRITTEN |  |

## Access and Security

Scope items: 21, 12, 62

| ID | Scope | Case | Type | Priority | Status | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| TC-021-10 | 21 | Role matrix: every endpoint run as every role and branch matches the scope in S02 | Integration | Must | NOT WRITTEN | Generated test |
| TC-021-11 | 21 | A staff member can never read or change another branch's data, including by guessing ids | Integration | Must | NOT WRITTEN |  |
| TC-012-05 | 12 | No endpoint exposes another guest's booking by reservation number or id | Integration | Must | NOT WRITTEN |  |
| TC-021-12 | 21 | API responses use DTOs; no password hashes, token hashes or internal fields appear | Integration | Must | NOT WRITTEN |  |
| TC-021-13 | 21 | Errors use the RFC 7807 shape with a code and request id; no stack traces | Integration | Must | NOT WRITTEN |  |
| TC-021-14 | 21 | Security headers are present: CSP, HSTS, no sniffing, Referrer-Policy no-referrer | Integration | Must | NOT WRITTEN |  |
| TC-021-15 | 21 | CORS allows known origins only | Integration | Must | NOT WRITTEN |  |
| TC-062-02 | 62 | Audit log rows cannot be updated or deleted, even directly in the database | Integration | Must | NOT WRITTEN |  |
| TC-062-03 | 62 | Reservations and payments cannot be deleted at the database level | Integration | Must | NOT WRITTEN |  |
| TC-021-16 | 21 | Logs contain no password, token or full card data; emails are masked where possible | Integration | Must | NOT WRITTEN |  |
| TC-021-17 | 21 | PostHog does not load until consent and never loads on /manage or /history | E2E | Should | NOT WRITTEN |  |
| TC-021-18 | 21 | Guest pages pass accessibility checks (WCAG 2.1 AA) | E2E | Should | NOT WRITTEN |  |
| TC-021-19 | 21 | Dependency scan passes in CI (Dependabot, npm audit, OWASP Dependency-Check) | Manual | Should | MANUAL-PENDING | CI job |

## Not Allowed (S02 cannot list)

Each case proves the action is refused. The role matrix test covers these too, but separate cases give readable failures.

| ID | Scope | Case | Type | Priority | Status | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| TC-X13-01 | not-allowed 13 | No Expired status exists; a reservation is created only after payment succeeds or the email code is accepted | Integration | Must | NOT WRITTEN | |
| TC-X16-01 | not-allowed 16 | A guest cannot sign in to the staff portal | Integration | Must | NOT WRITTEN | |
| TC-X17-01 | not-allowed 17 | A guest cannot choose a room number | Integration | Must | NOT WRITTEN | |
| TC-X19-01 | not-allowed 19 | A guest cannot change dates, guest count or extend a stay | Integration | Must | NOT WRITTEN | |
| TC-X23-01 | not-allowed 23 | A guest cannot pay by cash or bank transfer through the website | Integration | Must | NOT WRITTEN | |
| TC-X25-01 | not-allowed 25 | A Receptionist cannot approve refunds | Integration | Must | NOT WRITTEN | |
| TC-X26-01 | not-allowed 26 | A Receptionist cannot void charges or invoices | Integration | Must | NOT WRITTEN | |
| TC-X27-01 | not-allowed 27 | A Receptionist cannot change prices, tax rules, payment settings or branch policies | Integration | Must | NOT WRITTEN | |
| TC-X28-01 | not-allowed 28 | A Receptionist cannot manage employee accounts | Integration | Must | NOT WRITTEN | |
| TC-X29-01 | not-allowed 29 | A Receptionist cannot block rooms | Integration | Must | NOT WRITTEN | |
| TC-X30-01 | not-allowed 30 | A Receptionist cannot view the audit trail | Integration | Must | NOT WRITTEN | |
| TC-X31-01 | not-allowed 31 | Nobody can permanently delete reservations or payments | Integration | Must | NOT WRITTEN | |
| TC-X32-01 | not-allowed 32 | A Receptionist cannot approve a transfer to a different room type | Integration | Must | NOT WRITTEN | |
| TC-X33-01 | not-allowed 33 | A Receptionist or Supervisor cannot write off outstanding balances | Integration | Must | NOT WRITTEN | |
| TC-X34-01 | not-allowed 34 | A Receptionist cannot change dates of the same reservation more than 2 times | Integration | Must | NOT WRITTEN | |
| TC-X35-01 | not-allowed 35 | A Supervisor cannot approve refunds above the branch limit | Integration | Must | NOT WRITTEN | |
| TC-X36-01 | not-allowed 36 | A Branch Manager cannot create, edit or close branches | Integration | Must | NOT WRITTEN | |
| TC-X37-01 | not-allowed 37 | A Branch Manager cannot change global settings | Integration | Must | NOT WRITTEN | |
| TC-X38-01 | not-allowed 38 | A Branch Manager cannot manage other branches' employees or create Super Admins | Integration | Must | NOT WRITTEN | |
| TC-X39-01 | not-allowed 39 | A Branch Manager cannot manage the theme and amenity master lists | Integration | Must | NOT WRITTEN | |
| TC-X40-01 | not-allowed 40 | A Branch Manager cannot publish legal versions or handle data requests | Integration | Must | NOT WRITTEN | |
| TC-X41-01 | not-allowed 41 | Nobody can edit or delete audit log entries | Integration | Must | NOT WRITTEN | |
| TC-X43-01 | not-allowed 43 | A branch's currency cannot be changed after it has reservations | Integration | Must | NOT WRITTEN | |

## Overall Summary

| Group | Total | Not written | Manual pending | Passing | Failing |
| --- | --- | --- | --- | --- | --- |
| Guest: Search and Price | 14 | 14 | 0 | 0 | 0 |
| Guest: Booking and Payment | 22 | 22 | 0 | 0 | 0 |
| Guest: Manage, Cancel and History | 21 | 21 | 0 | 0 | 0 |
| Staff: Sign In and Accounts | 18 | 17 | 1 | 0 | 0 |
| Staff: Reservations and Changes | 21 | 21 | 0 | 0 | 0 |
| Staff: Check-in, Check-out and Rooms | 12 | 12 | 0 | 0 | 0 |
| Staff: Payments, Charges and Invoices | 9 | 9 | 0 | 0 | 0 |
| Supervisor | 10 | 10 | 0 | 0 | 0 |
| Branch Manager | 15 | 14 | 1 | 0 | 0 |
| Super Admin | 10 | 10 | 0 | 0 | 0 |
| Critical Scenarios (S13, all must pass) | 15 | 15 | 0 | 0 | 0 |
| Access and Security | 13 | 12 | 1 | 0 | 0 |
| Not Allowed (S02 cannot list) | 23 | 23 | 0 | 0 | 0 |
| **All** | **203** | **200** | **3** | **0** | **0** |

Last updated: 2026-11-21 (no tests written yet)
