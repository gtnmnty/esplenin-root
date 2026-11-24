---
name: test
description: Write and run tests for a feature so every scope item, critical scenario, and access rule is covered. Reports what is covered, what is not, and any real bugs the tests expose. Never changes production code.
---

A feature without tests works until the next change. This skill turns the plan and the scope into tests that prove the feature is correct, and shows what is still unproven.

Run it after building a feature and before `/review`.

## What This Skill Does Not Do

It does not change production code. If a test exposes a bug, the test stays failing and the bug is reported. Making a test pass by weakening it, or by editing the code under test, hides the problem instead of solving it.

## How to Invoke

```
/test [feature] [scope numbers]
```

Example: `/test password reset 12, 13`. If the feature or scope numbers are missing, infer them from the plan or the current branch. If still unclear, ask once.

---

## Step 1 — Establish What Must Be Proven

Read, in this order:

- The implementation plan from `/architect`, if one exists
- The scope items for this feature in the project documentation
- The critical scenarios list in the project documentation, if one exists
- `architecture.md` invariants and `code-standards.md` testing rules, including the coverage targets
- `test-cases.md` — the listed cases for this feature, found by scope number
- The feature's code

Write down every behavior that must be true. Each scope item needs at least one test. Each invariant touched by this feature needs at least one test.

Compare that list with `test-cases.md`:

- A behavior with no case: add a row to the right group in `test-cases.md`. Use the next free ID (`TC-[scope]-[nn]`), status `NOT WRITTEN`, and a priority. Do not remove or reword existing cases.
- A case with no matching behavior in the plan or scope: leave it and mention it in the report.
- If `test-cases.md` does not exist, say so and ask the developer to add it from the template before continuing.

## Step 2 — Learn the Project's Test Setup

Before writing anything, read the existing tests and test configuration.

- Which frameworks are used (for example JUnit 5, Mockito, Testcontainers, Vitest, Playwright)?
- Where do test files live, and how are they named?
- Are there shared fixtures, builders, or a controllable clock?

Match what exists. Do not introduce a new framework or pattern when one is already in use.

## Step 3 — Choose the Right Level

Pick the lowest level that can prove each behavior.

| Level | Use it for |
| --- | --- |
| Unit | Business rules in isolation: calculations, status changes, validation |
| Integration | The API with a real database (not a mock), migrations, constraints, transactions |
| End-to-end | The main user journey in a browser |
| Concurrency | Anything two users can do at once: the last item in stock, the same event arriving twice |

Always cover these, wherever the feature touches them:

1. **Happy path** — the normal case works.
2. **Rule edges** — every limit and boundary: just under, exactly at, just over.
3. **Failure paths** — invalid input, missing data, external service failing.
4. **Access** — for each role the project defines (and each owner, team, or tenant, if it has them), who is allowed and who is refused. One user must never read or change another user's or tenant's data.
5. **Time** — use a controllable clock, never real waiting. Test cutoffs in the correct timezone.
6. **Repeats** — the same request or event twice must not duplicate anything.
7. **Money** — if the feature handles money: amounts, rounding, refunds, and taxes use exact values.

## Step 4 — Write the Tests

Rules:

- One behavior per test. The test name states the behavior, not the method name.
- Start every test name with its case ID from `test-cases.md`, for example `TC-014-01 expired reset link is refused`. This is how cases are matched to tests.
- Write a test for every `NOT WRITTEN` case of this feature, `Must` first. If a case cannot be automated, set its type to `Manual` and its status to `MANUAL-PENDING`.
- Arrange, act, assert. No logic inside tests.
- Integration tests use a real database. Do not mock the database to prove database behavior.
- Do not test framework internals or getters and setters.
- Do not depend on test order or on data left by another test.
- Never put real secrets or personal data in fixtures.
- Put files where the project already puts them.

## Step 5 — Run Them

Run the new tests, then the full suite for the affected module. Run coverage if the project has it.

- Test passes: set the case to `PASSING` in `test-cases.md`.
- Test fails because the **test** is wrong: fix the test.
- Test fails because the **code** is wrong: leave it failing, set the case to `FAILING`, and report it.
- Never skip, disable, or loosen a test to get a green result.
- Never set `MANUAL-PASSED` yourself. Only the developer confirms a manual check.
- Update the feature summary and the Overall Summary tables in `test-cases.md` and the `Last updated` date. Change only Status, Notes, the summary tables, and new rows. Nothing else in the file.

## Step 6 — Report

```
## Test — [Feature Name]

### Scope coverage
| Scope item | Cases | Tests | Status |
| --- | --- | --- | --- |
| [number] [short name] | [case IDs] | [test names] | COVERED / PARTIAL / MISSING |

### test-cases.md changes
Added: [case IDs, or "none"]
Status changed: [case ID: old → new]
Still NOT WRITTEN: [case IDs, Must first]
Manual cases waiting for the developer: [case IDs]

### Tests added
[File] — [n] tests — [levels]

### Results
Passed: [n]   Failed: [n]   Skipped: [n]

### Failures that are real bugs
[Test name] — [file and line of the code at fault] — [what is wrong]
(or "None")

### Coverage
[Overall %] against target [x%]. [Critical flows %] against target [y%].
(or "No coverage tool configured")

### Not covered
[Behaviors that could not be tested and why]

### Summary
[X] of [Y] scope items covered. [n] real bugs found.
```

## Step 7 — Let the Developer Decide

After the report, stop. Do not fix production code. Do not suggest fixes unless asked.

Wait for the developer to:

- Ask you to fix a specific bug
- Tell you a failing test is wrong or out of scope
- Confirm the feature is covered and ready for `/review`

## The Standard

Tests exist to prove the feature is correct, not to make the build green. A suite that always passes proves nothing. A test that fails for a real reason is the most valuable output of this skill.
