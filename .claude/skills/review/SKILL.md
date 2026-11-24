---
name: review
description: Read-only review of the current branch's changes against main for correctness bugs, broken business rules, and access-control gaps. Reports severity-ranked, evidence-based findings. Never fixes. Run after /test and before /ship.
---

A review catches what tests miss: wrong logic, unhandled cases, and rules broken in code that the tests never reached. It looks at the change, not the whole feature. For a whole feature area, use `/audit`.

## What This Skill Does Not Do

- It never edits code, tests, or documents.
- It does not re-audit untouched code. Scope is the diff on this branch.
- It does not decide whether a finding is acceptable. It reports it.

## How to Invoke

```
/review
/review [branch or path]
```

If no target is given, review `git diff main` in the current repository. Run it per repository: the backend (`hotel-reservation/`) and frontend (`frontend/`) are separate git repos.

---

## Step 1 — Read the Rules

Read `architecture.md` (invariants, auth model) and `code-standards.md` (layering, error format, validation rules). These are the standards the diff is checked against.

## Step 2 — Read the Change

Run read-only commands: `git diff main --stat`, `git diff main`, `git log main..HEAD`. Read each changed file in full, not only the hunks, so the surrounding code is clear.

Stop and say so if there is no diff against `main`.

## Step 3 — Check

**Correctness**
- Logic errors, off-by-one, wrong comparisons, inverted conditions
- Unhandled null, empty, or error results
- Money and dates: integer minor units, branch timezone, UTC timestamps (invariant 8)
- Missing `@Transactional` boundaries where several writes must succeed or fail together

**Business rules**
- Each change against the invariants in `architecture.md`, especially: overbooking, reservations and payments never deleted, server-side price and refund calculation, webhook idempotency

**Access control**
- Every new or changed endpoint or service method enforces role and branch
- Ownership is checked before any mutation
- The client is never trusted for prices, totals, or refund amounts

**Standards**
- Layering: controller → service → repository; no entities returned from the API
- Errors go through the one exception handler with stable codes
- Input is validated at the edge

**Tests**
- New behavior has a test. If the diff adds a rule with no test, report it as a gap (point to `/test`)

## Step 4 — Verify Each Finding

Before recording a finding, confirm:

- You read the exact lines and the line numbers are right.
- The problem is real in context. A check may exist in another layer; if you cannot confirm, mark it `Unconfirmed`.
- It is not a duplicate. Merge repeats into one finding with every location.

Drop anything that fails these checks.

## Step 5 — Report

```
## Review — [Branch]

Compared: main...[branch], [n] files changed

### Findings
| ID | Severity | Location | Problem | Status |
| --- | --- | --- | --- | --- |
| C-1 / H-1 / M-1 / L-1 | Critical / High / Medium / Low | path:lines | [short problem] | Confirmed / Unconfirmed |

### Details
[One block per finding: what is wrong, why it matters, the evidence, and the smallest fix as a note. No full rewrites.]

### Gaps
[Changed behavior with no test]

### Summary
[n] findings: [n] Critical, [n] High, [n] Medium, [n] Low. [n] Unconfirmed.
```

Severity:
- **Critical** — wrong money, double booking, access bypass, data loss
- **High** — a business rule or invariant broken in a normal path
- **Medium** — wrong behavior in an edge case or a missing error path
- **Low** — naming, dead code, minor clarity

If there are no findings, say: "No issues found on this branch."

## Step 6 — Let the Developer Decide

Stop after the report. Do not fix anything. Wait for the developer to ask for a specific fix or accept the findings.

## The Standard

A review that finds nothing real is better than one that invents problems. Every finding points to a line, and every line reported is something the developer can check in a minute.
