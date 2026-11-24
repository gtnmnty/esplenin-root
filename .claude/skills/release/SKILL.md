---
name: release
description: Pre-deployment checklist for a release. Verifies the code, migrations, secrets, environments, and rollback plan, then gives a GO or NO-GO. Never deploys anything.
---

Deploying is the one step you cannot take back with a revert. This skill makes you answer the hard questions first, while it is still cheap.

Run it before tagging a release or deploying to staging or production.

## What This Skill Does Not Do

It does not deploy, tag, push, or run migrations. It does not read secret values. It gives a verdict, and the developer decides.

## How to Invoke

```
/release [target]
```

Target is `staging` or `production`. If missing, ask. Production is held to a higher standard than staging.

---

## Step 1 — Know What Is Shipping

- List the commits and merged PRs since the last release tag (`git log`).
- Read `progress-tracker.md` for open questions and known issues.
- Identify whether this release spans more than one repository. If it does, the dependency order matters (for example, backend before frontend).
- Propose the next version number using semantic versioning, with a short reason (breaking, feature, or fix).

## Step 2 — Run the Checklist

Mark each item **PASS**, **FAIL**, or **UNKNOWN**. UNKNOWN is never treated as PASS.

**Code**
- The release branch is `main` (or the tagged commit) and the working tree is clean
- CI is green for the exact commit (check with the CI tool if available, otherwise ask)
- `/review` and `/audit` were run for the features in this release, with no open Critical or High issues
- Coverage targets from `code-standards.md` are met

**Test cases**
- `test-cases.md` has cases for every scope item in this release
- No `Must` case for this release is `FAILING`, `NOT WRITTEN`, or `MANUAL-PENDING`. List every one that is. For production, any such case is a FAIL. For staging, list it as a risk
- `Last updated` in `test-cases.md` is not older than the newest commit that changed code or tests. If it is older, the statuses may be stale: mark this item UNKNOWN and tell the developer to run `/test`
- Every `Must` case marked `PASSING` was seen passing in the latest CI run (check with the CI tool if available, otherwise ask)

**Database**
- Every new migration was checked with `/migrate check`
- Migrations only move forward and no existing migration was edited
- Migrations are safe on production-sized data (no long locks, no unbackfilled `NOT NULL`)
- A recent backup exists and a restore path is known

**Configuration and secrets**
- Every environment variable the code reads exists in the target environment. Compare names only. Never read or print values
- No secrets in the repository or in build output
- Third-party keys point to the right mode for the target (for example test mode for staging)

**Environment**
- Versions match the pinned versions in `architecture.md`
- Container images build for the target architecture
- Health check and monitoring are in place for the new version
- Rate limits, CORS origins, and domains are correct for the target

**Verification**
- For production: this exact version ran on staging and the critical flows from the project documentation were checked
- List the critical flows to smoke test after deploy

**Rollback**
- Frontend: the previous build can be redeployed
- Backend: the previous image or version is known and available
- Database: because migrations only move forward, state what the plan is if a migration goes wrong
- Who decides to roll back, and what triggers it

## Step 3 — Report

```
## Release — [Version] to [Target]

### Shipping
[Commits, PRs, repositories, order of deploy]

### Checklist
| Area | Item | Result | Note |
| --- | --- | --- | --- |
| [Code/Database/...] | [item] | PASS / FAIL / UNKNOWN | [reason] |

### Test cases
Must cases failing: [IDs]
Must cases not written: [IDs]
Must cases waiting for a manual check: [IDs]

### Risks
[Anything that could go wrong, and what would show it]

### Rollback plan
[Concrete steps]

### Smoke tests after deploy
[List]

### Verdict
GO — all items pass.
or
NO-GO — [n] items failed or unknown: [list]
```

Production needs zero FAIL and zero UNKNOWN to be GO. Staging may be GO with UNKNOWN items if they are listed.

## Step 4 — Let the Developer Decide

Stop after the verdict. Do not tag, push, or deploy. Wait for the developer to resolve items, accept a risk, or say go.

## The Standard

A release is ready when you know how it will be verified and how it will be undone. If you cannot describe the rollback, you are not ready to deploy.
