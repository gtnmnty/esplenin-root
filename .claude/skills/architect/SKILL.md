---
name: architect
description: Plan a new feature or scope item before any code is written. Reads the scope, architecture, and standards, lists open decisions, and produces a step-by-step plan. Read-only on the project; writes no code and does not invent behavior. Run before /guide.
---

Building starts with the right decisions. This skill settles what a feature must do and how it fits the architecture, so `/guide` and the build have something firm to follow.

## What This Skill Does Not Do

- It writes no code and edits no project file except the open-questions list in `progress-tracker.md`, and only when the developer agrees.
- It does not invent product behavior. Anything the scope does not define is an open decision, not a guess.
- It does not change the schema file or other protected documents.

## How to Invoke

```
/architect [feature or scope numbers]
```

Example: `/architect 14, 15 password reset`. If the feature is missing, ask once.

---

## Step 1 — Load the Contract

Read, in this order:

- The scope items for this feature in the project documentation (`Esplenin_Hotel_Project_Docs_v0.3.pdf` at the project root holds the full document)
- `project-overview.md` for what is in and out of scope
- `architecture.md` for the stack, module boundaries, storage model, and invariants
- `code-standards.md` for layering, API, and security rules
- `progress-tracker.md` for decisions already made and open questions
- The existing code in the related module, so the plan fits what is there

If a scope item is not in the project documentation, say so. Do not plan it.

## Step 2 — Find the Decisions

For each scope item, list the decisions it needs:

- Behavior and rules: what happens, in what order, with which limits
- Access: which roles and branches can do it
- Data: which tables or columns change (for schema changes, name the `/migrate new` step)
- Money, time, and timezone rules
- Failure cases and what the user sees
- Cross-repo impact: backend first, then frontend

A decision the documents already settle is **Decided** and cites its source. A decision they do not settle is **Open**.

## Step 3 — Check the Plan Against the Invariants

For each planned change, check it against `architecture.md`. Any plan that would break an invariant is rejected and the alternative is stated. Examples to watch: overbooking, deleting reservations or payments, trusting client prices, storing raw card data.

## Step 4 — Write the Plan

Keep it short. Each step is one verifiable piece, in layer order (migration, service, controller, tests, then frontend).

```
## Architect — [Feature]

### Scope covered
[Scope numbers and one line each]

### Decided
- [Decision] — [source: document section or context file]

### Open
- [Question] — [why it matters, what it blocks]

### Plan
1. [Step] — [module or file area] — [done when …]
2. [Step]

### Split
[If the feature spans more than one system boundary or repo, list the parts. Build the first part only.]

### Risks
[What could go wrong and how it would show]
```

If there are open decisions that block the plan, stop after the report and say so. Do not build the plan with a guess inside it.

## Step 5 — Record Open Questions

Ask the developer whether to add the **Open** items to `progress-tracker.md` as open questions. Add them only after a yes.

## Step 6 — Let the Developer Decide

Stop after the report. Wait for the developer to answer the open questions or approve the plan. The next step is `/guide`, which explains how to build each step.

## The Standard

A feature is ready to build when every step has a clear result and every decision is either settled by the documents or written down as open. If a step depends on a guess, it is not ready.
