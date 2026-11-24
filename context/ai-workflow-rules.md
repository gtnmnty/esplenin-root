# AI Workflow Rules

## Approach

Build Esplenin incrementally with a spec-driven workflow. The Esplenin Project Documentation (v0.4) and these context files define what to build, how to build it, and the current state. Implement against them. Do not infer or invent behavior.

If two sources disagree, the document says which wins:
- Numbers and limits: S08 Default Values
- Behavior and permissions: S02 Scope, then S03, then S08 rules
- Pages and screens: S04
- Technology choices: S05
- Database tables and columns: the schema file (`hotel_schema_v4.1.md`)

## Scoping Rules

- Work on one feature unit at a time, one feature per git branch
- Prefer small, verifiable increments over large speculative changes
- Do not combine unrelated system boundaries in a single implementation step
- Build the core flow first. Cache, rate limiting, NGINX layers and the monitoring stack come after it works
- Follow the phase order in S14 (Phase 0 Foundation → Phase 6 Hardening)

## When to Split Work

Split an implementation step if it combines:

- Backend and frontend changes for different features (the frontend merges after the backend)
- Database migrations and unrelated business logic
- Payment/refund logic and unrelated UI work
- More than one S02 scope area (for example guest booking and front desk check-in)
- Behavior not clearly defined in the documentation

If a change cannot be verified end to end quickly, the scope is too broad. Split it.

## Handling Missing Requirements

- Do not invent product behavior not defined in the documentation
- If a requirement is ambiguous, resolve it in the relevant context file (and the project documentation) before implementing
- If a requirement is missing, add it as an open question in `progress-tracker.md` before continuing
- Scope changes need an update to S02 first. S02 is the contract

## Protected Files

Do not modify the following unless explicitly instructed:

- `components/ui/*` (generated shadcn/ui components)
- `hotel_schema_v4.1.md` (stays unchanged until Monty says go; S09 in the project documentation v0.4 matches it)
- Flyway migrations that have already run (fix mistakes with a new migration)
- `.env` files and any secrets
- Lock files (change only through the package manager)
- Third-party library internals

## Keeping Docs in Sync

Update the relevant context file whenever implementation changes:

- System architecture or boundaries
- Storage model decisions
- Code conventions or standards
- Feature scope (update S02 first)
- Status of work in `progress-tracker.md`

When behavior changes, the project documentation and the schema file must be updated too (Definition of Done).

## Before Moving to the Next Unit

1. The current unit works end to end within its defined scope and the S02 scope numbers it covers are listed
2. No invariant defined in `architecture.md` was violated
3. Tests are written and pass: backend `./mvnw verify` (in `hotel-reservation/`, Docker required for Testcontainers), frontend `pnpm build` and `pnpm lint`. The frontend has no test script yet; add one (Vitest/Playwright per `code-standards.md`) before relying on it. The unit's cases in `test-cases.md` are listed, and no `Must` case is failing or not written
4. Access rules checked for every role and branch
5. Audit log entries, error states, empty states and translatable texts are in place
6. `progress-tracker.md` reflects the completed work
7. Run `/review` before moving on, and `/audit` when a feature area is complete
8. Run `/trace` to confirm every claimed S02 scope item has code and tests, and `/i18n-check` for any UI, email, or error text change
9. Run `/docs-sync` after the change so `progress-tracker.md` and the context files match the code

## Session Routine

- Start: read `progress-tracker.md`; for a fresh project, run `/kickoff`
- New feature: run `/architect` to settle the decisions and plan, then `/guide` to learn how to build each step. Read `ui-context.md` first for UI work
- Database changes: run `/migrate new` to draft, then `/migrate check` before merging
- After building: run `/trace` (scope coverage), `/test` (tests), then `/review` (diff bugs)
- Feature area complete: run `/audit`
- UI work: run `/imprint` after each component so it saves its patterns to `ui-registry.md`
- Something breaks: run `/recover` to diagnose the failure type before fixing
- Session start and end: run `/remember restore` at the start and `/remember save` at the end
- Something breaks: diagnose before fixing; state the root cause first
- Before a PR: run `/commit` to save task-scoped commits, then `/ship` to check the branch and draft the PR description
- Before a release: run `/release` for GO or NO-GO

Project skills live in `.claude/skills/<name>/SKILL.md` and run as slash commands.
