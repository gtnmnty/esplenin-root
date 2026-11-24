---
name: commit
description: Group changed files by task, create or switch to the task's branch, stage only that task's files, commit, and push. Never touches main. Shows the plan and waits for confirmation before staging or pushing anything.
---

One task, one branch, one clean commit. This skill stops unrelated files from sneaking into a commit, and stops work from piling up on the wrong branch.

Run it whenever a piece of work is ready to be saved. It can run many times per task.

## What This Skill Does Not Do

- It never commits to `main` and never pushes to `main`.
- It never force-pushes, rebases, resets history, or deletes branches.
- It never stages with `git add .` or `git add -A`. Files are staged by exact path.
- It does not open pull requests or merge. That is for `/ship` and the developer.
- It does not fix code. If it finds a problem, it reports it.
- It never skips the confirmation step.

## How to Invoke

```
/commit
/commit [task name]
```

If a task name is given, use it. Otherwise work the task out in Step 4. It works on one repository at a time.

---

## Step 1 — Read the Rules

Read:

- The git rules in `code-standards.md`: branch types and naming, commit format, forbidden words
- `ai-workflow-rules.md`: protected files, and any folder-to-feature table
- `architecture.md`: the module list and folders
- `progress-tracker.md`: the Current Goal and Next Up

If the branch or commit rules are missing, ask the developer for them before continuing.

## Step 2 — Read the Changes

Run read-only commands: `git status`, `git diff`, `git diff --staged`, `git branch --show-current`, and list untracked files. Run `git fetch` to see the remote state.

Stop and say so if:

- There are no changes.
- HEAD is detached.
- There are unresolved merge conflicts.

Note any files that are already staged. They are part of the plan and may be moved or unstaged after confirmation.

## Step 3 — Safety Checks

**Blockers** (stop and report, change nothing):

- A secret is in a changed file: API key, token, password, connection string, private key. Show file and line only, with the value redacted (`sk-****`).
- `.env` or another secret file is changed or untracked and not ignored.
- Conflict markers are in a changed file.

**Warnings** (report in the plan):

- A protected file was modified.
- Build output, editor settings, logs, or large binaries are among the changes.
- A migration file is included. Suggest `/migrate check`.
- A lockfile changed with no dependency file change.

## Step 4 — Group the Files by Task

**Map each file to a module.** Backend files map by module or package. Frontend files map by route or feature folder. Use `architecture.md` and any folder-to-feature table.

**Decide the task.**

- If the Current Goal in `progress-tracker.md` is specific and the files match it, use it. Do not ask.
- Ask exactly one short question when:
  - the Current Goal is missing or too broad ("start Phase 0"),
  - the files and the goal disagree,
  - or the files in one module clearly belong to different tasks.
- If the changes cover several tasks, make one plan per task.

**Placement rules.**

- Tests go with the code they test.
- A migration goes with the feature that needs it.
- A shared file (component, utility, config) goes with the task that needs it. If no task needs it, ask.
- A dependency file changed because a feature needs a new library goes with that feature. A dependency file changed alone goes to a `chore/` branch.
- CI, Docker, and tool configuration go to a `chore/` branch.
- Context files, README, and project docs go to a `docs/` branch.
- A bug fix inside one module goes to a `fix/` branch.
- A file that mixes two tasks cannot be split by this skill. Ask which branch it goes to, and say that the other task's lines will travel with it.
- Never guess. A file that fits nowhere is left out and listed in the plan.

## Step 5 — Choose the Branch

Branch name format, from the project's rules:

```
type/id-short-task
```

- `type` is `feature`, `fix`, `chore`, or `docs`.
- `id` is the issue number if the project tracks tasks as issues, otherwise the main scope number. If neither exists, leave it out.
- `short-task` is two to four words, lowercase, hyphen-separated, under about 40 characters in total.
- Examples: `feature/12-staff-signin`, `fix/21-lockout-timer`, `chore/update-dependencies`, `docs/update-context-files`.
- For a feature that spans the backend and frontend repositories, use the same branch name in both.

**Which branch.**

| Situation | Plan |
| --- | --- |
| Already on the task's branch | Stay on it |
| The task's branch exists locally or on the remote | Switch to it. If it is behind `main`, say so. Do not rebase |
| The task's branch does not exist | Create it from an up-to-date `main` |
| On `main` with changes | Create the task branch from `main`, carrying the changes. Never commit on `main` |
| On another task's branch with changes for a different task | Create the new task's branch from `main`, carrying only that task's changes |

If the changes cannot be carried to the new branch without a conflict, stop and report.

## Step 6 — Write the Commit Message

Follow the project's format exactly. The usual shape:

```
type(scope): short summary
```

- Imperative mood ("add", not "added").
- Within the project's length limit.
- Avoid words the project forbids.
- Add a body only to explain why.
- One concern per commit. If a task mixes concerns (for example a migration and a service change), propose separate commits and list the files for each.

## Step 7 — Show the Plan and Wait

```
## Commit plan — [n] branch(es)

### 1. [branch name] — [new from main / exists / current]
Files:
- [path]
- [path]
Commit: [message]

### Left out
- [path] — [reason, or the question that decides it]

### Checks
Secrets: PASS | BLOCKER
Warnings: [list or "none"]

Confirm? (yes / change / skip a branch)
```

Then stop. Do nothing until the developer answers.

## Step 8 — Execute After Confirmation

For each confirmed plan, in order:

1. If the branch is new: update `main` with a fast-forward only, then create the branch from it.
2. If the branch exists: switch to it.
3. If any files are already staged that are not in the plan, unstage them.
4. Stage the planned files by exact path.
5. Run `git diff --staged --stat`. Confirm it lists only the planned files. If it does not, unstage and stop.
6. Commit with the confirmed message.
7. Push. Use upstream tracking for a new branch.

If a push is rejected, stop and report. Never force-push. Never rebase on your own.

After the last plan, run `git status` and confirm that only the left-out files remain.

## Step 9 — Report and Stop

```
Committed and pushed:
- [branch] — [commit message] — [n] files

Still uncommitted:
- [path] — [why]

Next: keep building, or run /ship when the task is finished.
```

Do not open a pull request. Do not merge.

## The Standard

Every commit contains one task and nothing else. Anyone reading the branch list and the commit history should see the project's work as a clean series of small, finished tasks.
