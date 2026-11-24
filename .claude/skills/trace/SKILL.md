---
name: trace
description: Map a finished feature to the scope items it claims to cover. Shows which scope items have code, which have tests, and which code belongs to no scope item. Reports only, never fixes.
---

Scope is the contract. A feature can be "done" and still skip half of what it promised, or quietly add something nobody asked for. This skill checks the code against the contract, item by item.

Run it after building a feature, before `/test` and `/review`.

## What This Skill Does Not Do

It does not write code or tests, and it does not decide what the scope should be. It reports what exists and what does not.

## How to Invoke

```
/trace [scope numbers or feature name]
```

Example: `/trace 14, 26` or `/trace password reset`. If nothing is given, infer the scope items from the current branch, the implementation plan, or `progress-tracker.md`. If still unclear, ask once.

---

## Step 1 — Load the Contract

Read, in this order:

- The scope list in the project documentation (the numbered items)
- The implementation plan from `/architect`, if one exists
- `project-overview.md` for what is in and out of scope
- `test-cases.md` — the cases listed for these scope items

For each requested scope item, write down the exact behavior it requires, including limits and role restrictions. If an item is vague, record it as **Unclear**. Do not guess its meaning.

## Step 2 — Find the Evidence

For every scope item, search the code. Evidence means a real file and line numbers.

Look for:

- **Implementation** — the service, endpoint, or screen that performs the behavior
- **Rule enforcement** — the role, branch, limit, or validation the item requires
- **Tests** — at least one test that exercises the behavior. Match tests to cases in `test-cases.md` by the case ID at the start of the test name
- **Cases** — the case IDs in `test-cases.md` for this scope item, with their status
- **User-facing pieces** — the page, form, email, or notification the item names

Rules:

- Only count what you can point to. A function name that sounds right is not evidence.
- Do not trust the Status column in `test-cases.md`. Check that a test with that case ID really exists. A case marked `PASSING` with no matching test is a finding.
- A scope item with no case in `test-cases.md` is a finding, even if tests exist.
- If you are unsure whether code satisfies an item, mark it **Unconfirmed** and say what you could not verify.
- Read the code. Do not judge from file names.

## Step 3 — Find the Strays

Compare in the other direction. List code added on this branch (`git diff main`) that does not map to any scope item: new endpoints, new fields, new screens, new settings.

Each stray is either scope creep, a missing scope item, or a supporting detail. Do not decide which. Report it.

## Step 4 — Report

```
## Trace — [Feature Name]

### Scope coverage
| Scope item | Requirement | Code | Cases | Tests | Status |
| --- | --- | --- | --- | --- | --- |
| [number] | [short requirement] | [file:lines] | [case IDs or "none"] | [test names or "none"] | DONE / PARTIAL / MISSING / UNCONFIRMED |

### Gaps
[Each PARTIAL or MISSING item: what exactly is absent]

### Untested
[Scope items with code but no test]

### Test case gaps
Scope items with no case in `test-cases.md`: [numbers]
Cases with no matching test: [case IDs, Must first]
Cases marked PASSING with no matching test: [case IDs]
Cases that are FAILING: [case IDs]

### Strays (code with no scope item)
[file:lines — what it does]

### Unclear scope items
[Items too vague to verify]

### Summary
[X] of [Y] scope items done. [Z] untested. [N] strays.
```

Use the same numbers as the documentation so the developer can paste the result into a PR.

## Step 5 — Let the Developer Decide

After the report, stop. Do not fix anything.

Wait for the developer to:

- Ask you to close a specific gap
- Say a stray is intentional (then it needs a scope entry)
- Say an item is deferred (then it goes in `progress-tracker.md`)

## The Standard

Every promised behavior has code and a test. Every piece of code traces to a promise. Where those two lists disagree, the developer needs to know before moving on.
