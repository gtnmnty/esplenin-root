---
name: docs-sync
description: After a change, find where the documentation no longer matches the code, report the drift, and update the context files only after approval. Never edits protected documents without an explicit go.
---

Docs that lie are worse than no docs. The next session, human or AI, will trust them. This skill finds where the code and the documents have drifted apart.

Run it after a feature is reviewed, or whenever a decision changed during building.

## What This Skill Does Not Do

- It does not change code to match the docs. If the docs are right and the code is wrong, that is a bug, and it is reported as one.
- It does not edit protected documents (for example the database schema file) without an explicit go from the developer.
- It does not rewrite documents. Edits are small and targeted.

## How to Invoke

```
/docs-sync
```

---

## Step 1 — Find What Changed

Use `git diff main` and `git log` to list what changed on this branch: new modules, endpoints, tables, settings, dependencies, folders, UI tokens, and rules. Read the implementation plan and `progress-tracker.md` for decisions made along the way.

## Step 2 — Compare Against Each Document

Check each document for statements that are now false, missing, or out of date.

| Document | Check |
| --- | --- |
| `architecture.md` | Stack and versions, module boundaries, storage model, auth model, invariants |
| `code-standards.md` | Conventions the new code introduced or broke, folder layout |
| `ui-context.md` | New tokens, components, layout patterns, hardcoded values |
| `project-overview.md` | Features, scope, success criteria |
| `ai-workflow-rules.md` | Protected files, commands, checklists |
| `progress-tracker.md` | Completed work, next up, open questions, decisions with reasons |
| Project documentation | Scope items, business rules, defaults, status flows |
| Schema file | Tables, columns, constraints versus the migrations |

Rules:

- Evidence only. Point to the code line and the document line that disagree.
- If you cannot tell whether the document or the code is right, mark it **Unclear** and ask.
- A behavior the code has and no document mentions is **Undocumented**.
- A statement in a document the code no longer follows is **Stale**.

## Step 3 — Report

```
## Docs Sync — [Branch or Feature]

### Drift found
| Document | Section | Problem | Evidence | Type |
| --- | --- | --- | --- | --- |
| [file] | [heading] | [what is wrong] | [code file:line vs doc line] | STALE / UNDOCUMENTED / UNCLEAR |

### Protected documents
[Drift in documents that need an explicit go, listed separately]

### Possible code bugs
[Places where the document looks right and the code looks wrong]

### Summary
[X] drift items in [Y] documents.
```

If there is no drift, say: "No drift found. Documents match the code."

## Step 4 — Wait for Approval

Stop after the report. Ask the developer which items to apply. Do not edit anything yet.

## Step 5 — Apply Approved Edits

For each approved item:

- Make the smallest edit that fixes it. Do not reword unrelated text.
- Keep each document's existing format.
- Record decisions that affect design in `progress-tracker.md` under Architecture Decisions, with the reason.
- Update Completed, In Progress, and Next Up in `progress-tracker.md`.
- Edit protected documents only if the developer said go for that specific document.

Then list exactly what changed:

```
Updated:
- [file] — [section] — [what changed]
```

## The Standard

A fresh reader who has only the documents should describe the system exactly as the code behaves. Where they cannot, the documents are wrong, and this skill finds where.
