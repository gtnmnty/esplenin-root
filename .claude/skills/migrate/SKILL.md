---
name: migrate
description: Check database migrations for safety and consistency, or draft a new migration. Catches edited migrations, number conflicts, destructive changes, and drift from the schema file. Never runs a migration against a real database.
---

A bad migration cannot be undone by reverting code. It runs once, on real data, and the history is permanent. This skill slows that step down on purpose.

Run `/migrate check` before merging any branch that touches migrations. Run `/migrate new` when a feature needs a schema change.

## What This Skill Does Not Do

- It does not run migrations against a staging or production database. Ever.
- It does not edit a migration that has already run. Mistakes are fixed with a new migration.
- It does not edit the schema document unless the developer says go. The schema file is protected.

## How to Invoke

```
/migrate check
/migrate new [description]
```

If neither is given, ask which one is needed.

---

## Check Mode — `/migrate check`

### Step 1 — Gather

- Find the migration folder (for example `db/migration`) and the migration tool in use (for example Flyway).
- List every migration in order.
- Read the schema document, `architecture.md` invariants, and the plan or scope for the current branch.
- Use `git log` and `git diff main` to see which migrations are new on this branch and which already exist on main.

### Step 2 — Check

**Order and naming**
- Version numbers are unique and have no unexplained gaps.
- Names follow the tool's pattern (for example `V12__add_orders_table.sql`) and describe the change.
- If another branch added the same number, the later branch must rename its file after rebasing.

**History is untouched**
- No migration that already exists on main was modified, renamed, or deleted. This is the most serious finding. Mark it **Critical**.

**Safety**
- Flag every destructive or risky statement: `DROP`, `TRUNCATE`, `DELETE` without a precise `WHERE`, column rename, type change, shrinking a column, adding `NOT NULL` without a default or backfill, adding a constraint to a table that already holds data, `CASCADE` deletes.
- Each migration does one thing. Mixed schema changes and data changes are flagged.
- New constraints and indexes have explicit names.
- Large-table changes that would lock the table are flagged.

**Invariants**
- Compare against `architecture.md`. Any migration that weakens a rule (for example allowing deletes on records that must never be deleted, or removing a uniqueness or exclusion constraint) is **Critical**.

**Consistency with the schema document**
- Every column, table, and constraint the migration adds, changes, or removes should match the schema document, or be listed as a pending change. List every difference.

**Runnable**
- Confirm it applies cleanly to an empty database (for example with the project's integration test setup). If that cannot be run, say so. Do not claim it works.

### Step 3 — Report

```
## Migration Check — [Branch or Feature]

### Order and naming
[PASS / ISSUES FOUND]

### History untouched
[PASS / ISSUES FOUND]

### Safety
[PASS / ISSUES FOUND]
[Each risky statement: file, line, why it is risky]

### Invariants
[PASS / ISSUES FOUND]

### Schema document drift
[PASS / ISSUES FOUND]
[Each difference]

### Runs on an empty database
[PASS / FAIL / NOT RUN — reason]

### Summary
[X] issues found. [Y] Critical.
```

Severity: **Critical** (edited history, weakened invariant, data loss), **Important** (risky statement, drift, unnamed constraint), **Minor** (naming, style).

### Step 4 — Let the Developer Decide

Stop after the report. Do not fix anything unless asked.

---

## New Mode — `/migrate new [description]`

### Step 1 — Understand the Change

Read the plan, the schema document, and the latest migrations. If the change is ambiguous, ask one question. Do not guess at column types, nullability, or deletion behavior.

### Step 2 — Draft the Migration

- Use the next free version number. Check main and other branches for conflicts.
- Name it after what it does.
- One concern per file.
- Name every constraint and index explicitly.
- For a new `NOT NULL` column on a table with data: add it nullable, backfill, then add the constraint, in separate steps or files.
- Never touch an existing migration file.
- No secrets, no environment-specific values.

### Step 3 — Show, Do Not Run

Present the file path and the full SQL, plus a short list of what changes and anything risky. State what the schema document needs updated, as a note for the developer.

Do not execute the migration. Do not edit the schema document.

### Step 4 — Let the Developer Decide

Wait for the developer to approve the migration, change it, or reject it. Once approved, offer to run `/migrate check` and `/test` on it.

## The Standard

Migrations move forward only. A migration that has run is history, and history is never edited. If it was wrong, write a new one.
