---
name: ship
description: Check that a pushed task branch is ready to merge and write its pull request description. Reports blockers and unknowns, then drafts the PR. Never commits, pushes, or merges. Use /commit to save work.
---

Finishing a task is more than pushing code. This skill answers two questions: is this branch ready to merge, and what should the pull request say?

Run it once per task, after the work is committed and pushed with `/commit`.

## What This Skill Does Not Do

- It does not stage, commit, or push. That is `/commit`.
- It never merges, force-pushes, or deletes branches.
- It does not fix code. If it finds a problem, it reports it.
- It does not invent test results or scope numbers.

## How to Invoke

```
/ship
```

---

## Step 1 — Read the Rules

Read the pull request rules in `code-standards.md` and `ai-workflow-rules.md`: required sections, commit and title format, merge method, protected files. Read `progress-tracker.md` and `test-cases.md`. If the PR rules are missing, say so and ask the developer for the format before continuing.

## Step 2 — Know What Is Shipping

Run read-only commands:

- `git branch --show-current`
- `git log main..HEAD` for the commits on this branch
- `git diff main --stat` for the files changed
- `git status`, and compare the branch with its remote

Work out the scope numbers this branch covers from the branch name, the commit messages, the implementation plan, and `/trace` output if it exists. If you cannot tell, ask once.

## Step 3 — Run the Pre-Merge Checks

Mark each item **PASS**, **FAIL**, or **UNKNOWN**. UNKNOWN is never treated as PASS.

**Branch**
- The current branch is not `main`
- There are no uncommitted changes. If there are, say: "Run `/commit` first."
- There are no unpushed commits. If there are, say: "Run `/commit` first."
- The branch is up to date with `main`, or the difference is listed. If it is behind, say so. Do not rebase
- No merge conflicts with `main`
- The diff contains no secret. Show file and line only, with the value redacted. Any secret is a **FAIL**
- No protected file was modified without the developer's go

**Quality**
- Tests were run and pass. Check CI for the pushed commit if a CI tool is available, otherwise ask
- No `Must` case in `test-cases.md` for these scope items is `FAILING`, `NOT WRITTEN`, or `MANUAL-PENDING`. List any that are
- `/trace` was run and every scope item for this branch is covered
- `/review` was run with no open Critical issues
- If migrations are included: `/migrate check` was run
- If user-facing text changed: `/i18n-check` was run
- Documentation is in sync (`/docs-sync`), or the drift is listed
- For UI changes: screenshots exist

## Step 4 — Draft the Pull Request

**Title.** Use the project's commit format, because a squash merge uses the title as the commit message:

```
type(scope): short summary
```

**Description.** Fill only what you can verify from the diff, the commits, the tests, and the tracker. Mark anything you could not verify as `[confirm]`.

```
## Summary
[What this change does and why, in two or three sentences]

## Scope covered
[Scope item numbers, from /trace]

## Related issue
[Closes #n, or "None"]

## How it was tested
[Tests added, test case IDs, commands run, results]

## Database changes
[Migration files, or "None"]

## Screenshots
[Required for UI changes, or "No UI changes"]

## Docs updated
[Context files, project documentation, schema file, or "None needed"]

## Notes
[Cross-repo dependencies (which repository merges first), risks, follow-ups]
```

## Step 5 — Report

```
## Ship — [Branch Name]

### Shipping
[n] commits, [n] files, scope [numbers]

### Pre-merge checks
| Area | Item | Result | Note |
| --- | --- | --- | --- |
| [Branch/Quality] | [item] | PASS / FAIL / UNKNOWN | [reason] |

### Pull request
Title: [title]

[Description]

### Verdict
READY — all checks pass.
or
NOT READY — [n] items failed or unknown: [list]
```

## Step 6 — Let the Developer Decide

Stop after the report. Wait for the developer to:

- Fix the failed items and run `/ship` again
- Accept a risk and say go
- Ask you to open the pull request, if a tool for that is available

Only after a separate, explicit ask, open the pull request. Never merge.

After the developer squash-merges, the branch is deleted and the next task starts from an up-to-date `main`.

## The Standard

A branch is ready to merge when you can say what it does, which requirements it covers, and how you know it works. If any of those is a guess, it is not ready.
