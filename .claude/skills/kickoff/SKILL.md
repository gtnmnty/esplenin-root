---
name: kickoff
description: Before the first line of code on a project, check that the context files are filled in, git and secrets are set up, and the project is ready to build. Reports blockers and warnings. Does not fix anything.
---

Most AI-assisted projects go wrong in the first hour, not the last. The context files are blank, secrets sit in the repo, or nobody knows which branch to work on. This skill catches that before any code exists.

Run it once at project start, and again after a long break.

## What This Skill Does Not Do

It does not write code, fill in context files, or change settings. It reports what is missing and lets the developer decide.

## How to Invoke

```
/kickoff
```

---

## Step 1 — Find the Context Files

Look in the project root, then `context/`, then `docs/`. Expect these seven:

- `project-overview.md`
- `architecture.md`
- `code-standards.md`
- `ui-context.md` (skip if the project has no UI)
- `ai-workflow-rules.md`
- `progress-tracker.md`
- `test-cases.md`

If a file is missing, record it as a **Blocker**. If the files are split across several folders, record that as a **Warning**.

## Step 2 — Check the Context Files Are Real

Read each file fully, then check:

1. **Placeholders.** Search for unfilled template text: `[e.g.`, `[Project Name]`, `[Feature`, `[Rule`, `[Role]`, `[folder]`, and similar bracketed prompts. Each one is a **Blocker**.
2. **TBD values.** A `[TBD]` is allowed only if the same item is listed under Open Questions in `progress-tracker.md`. A `[TBD]` that is not listed there is a **Warning**.
3. **Contradictions.** Compare files with each other. Examples: the stack in `architecture.md` versus the technology in `code-standards.md`; build commands in `ai-workflow-rules.md` versus the project's real scripts. Each contradiction is a **Warning**.
4. **Tracker.** `progress-tracker.md` must have a Current Goal and at least one Next Up item. If not, it is a **Blocker**.
5. **Invariants.** `architecture.md` must list at least three. If not, it is a **Warning**.
6. **Test cases.** `test-cases.md` must have at least one real case (not a `[placeholder]` row) for the first feature in `progress-tracker.md` Next Up. If not, it is a **Warning**. Also check that case IDs are unique, every case has a scope number, and every case has a priority. Each problem is a **Warning**.

## Step 3 — Check the Project Setup

Run read-only commands only.

- **Git.** Is this a git repository? Is there a remote? Is the working tree clean? Is the current branch `main` (work should happen on a feature branch)? A missing repository is a **Blocker**. The rest are **Warnings**.
- **Secrets.** Is `.env` listed in `.gitignore`? Is any `.env` file tracked by git? Search tracked files for obvious secrets (API keys, tokens, passwords, connection strings). A tracked secret is a **Blocker**. Never print the secret value. Show the file and line only, with the value redacted (`sk-****`).
- **Manifests.** Find files such as `pom.xml`, `package.json`, `requirements.txt`. Compare versions with the stack in `architecture.md`. A mismatch is a **Warning**.
- **Build and test commands.** Confirm the commands named in `ai-workflow-rules.md` exist in the project. If the project has no code yet, say so and skip this check.
- **Reports folder.** Does `reports/` exist at the project root, outside the source folders? If not, it is a **Warning**. Offer to create it.
- **Lockfiles and pinned versions.** If manifests exist, are lockfiles committed? If not, it is a **Warning**.

## Step 4 — Report

Do not soften findings. Report honestly.

```
## Kickoff — [Project Name]

### Context files
[PASS / ISSUES FOUND]
[List each problem with its severity and file]

### Project setup
[PASS / ISSUES FOUND]
[List each problem with its severity]

### Summary
Blockers: [n]
Warnings: [n]

[If no blockers: "READY — nothing blocks you from starting."]
[If blockers: "NOT READY — resolve the blockers first."]
```

Severity:

- **Blocker** — starting now would cause rework or risk (blank context files, tracked secrets, no git).
- **Warning** — fix soon, but it does not stop work (unlisted TBDs, minor mismatches, missing `reports/`).

## Step 5 — Let the Developer Decide

After the report, stop. Do not fix anything. Do not suggest fixes unless asked.

Wait for the developer to:

- Ask you to fix a specific item
- Say an item is intentional
- Confirm everything is resolved, then re-run `/kickoff`

## The Standard

A project is ready when anyone, including a fresh AI session, can read the context files and know what to build, how to build it, and where things stand. If a fresh reader would have to guess, the project is not ready.
