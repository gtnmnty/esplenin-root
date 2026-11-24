---
name: audit
description: Read-only code audit of a feature's source folder covering security, data leaks, functionality, maintainability, and readability. Saves a severity-ranked, numbered Markdown report to the reports folder, named after the feature. Use when the developer runs /audit or asks for a code, security, or quality audit.
---

An audit finds real problems in real code. It never changes the code it audits, and it never guesses.

## How to Invoke

```
/audit [SOURCE_FOLDER] [FEATURE_NAME]
```

- `SOURCE_FOLDER` — the folder to audit. Required. If missing, ask for it.
- `FEATURE_NAME` — what is being audited, e.g. `auth email service`. Optional. If missing, infer it from the folder name and code. If it is still unclear, ask once.

The report always goes in `reports/` at the project root. Create the folder if it does not exist. If `reports/` is inside `SOURCE_FOLDER`, stop and ask for a different location.

---

## Rules

1. **Read-only on source.** Never modify, create, or delete any file inside `SOURCE_FOLDER`.
2. **One output file.** The only file you may create is the report, inside `reports/`. Never overwrite an existing report.
3. **Evidence only.** Report only issues you can point to in the actual code, with a real file path and line numbers. No invented issues. No generic advice.
4. **Be honest about uncertainty.** If you are not sure something is an issue, mark it `Unconfirmed` and state what you could not verify. Never present a guess as fact.
5. **No secrets in the report.** If you find a credential, show the file and line but redact the value (for example `sk-****`).

---

## Step 1 — Name the Report

File name format:

```
NN-feature-name-audit.md
```

- `NN` — two-digit sequence number. List `reports/`, find the highest existing `NN-` prefix, and add 1. If none exist, use `01`.
- `feature-name` — the feature in lowercase, words joined by hyphens. Letters, digits, and hyphens only. No spaces, colons, or `#`.
- Keep it under 40 characters. Drop filler words.

Examples:

- `01-auth-email-service-audit.md`
- `02-gift-cards-audit.md`
- `03-order-checkout-audit.md`

## Step 2 — Map the Code

1. List all files in `SOURCE_FOLDER`.
2. Skip generated and vendored content: `node_modules`, `target`, `build`, `dist`, `.git`, lockfile contents, binaries, and minified files.
3. Identify the language, framework, entry points, config files, and dependency manifests (`pom.xml`, `package.json`, `requirements.txt`, etc.).
4. Note the project's existing patterns (naming, error handling, response objects). Fixes must match them.

## Step 3 — Read and Check

Read every relevant file fully. Do not judge from file names.

**1. Security**
- Hardcoded secrets or credentials
- Weak or missing authentication and authorization (including missing ownership checks)
- Missing input validation
- Injection: SQL, command, XSS
- Insecure configuration: CORS, CSRF, debug mode, exposed admin or actuator endpoints
- Weak password or token handling
- Vulnerable or outdated dependencies — flag only versions you are confident are vulnerable; mark the rest `Unconfirmed`

**2. Data leaks**
- Sensitive data in API responses, logs, error messages, or stack traces
- Internal models or entities returned directly instead of safe response objects
- Personal data stored or transmitted without protection

**3. Functionality**
- Bugs, unhandled errors, null values
- Race conditions
- Incorrect business logic
- Missing transactions
- Inefficient queries or loops (such as N+1)
- Resource leaks

**4. Maintainability**
- Duplicated code
- Oversized files or functions
- Tight coupling, missing separation of concerns
- Hardcoded values that belong in configuration
- Missing error handling strategy

**5. Readability**
- Unclear naming
- Misleading or missing comments
- Inconsistent style
- Dead or unused code

## Step 4 — Verify Each Finding

Before recording an issue, confirm:

- You opened the file and saw the exact lines.
- The line numbers are correct.
- The problem is real in context (a check may exist in another layer). If you could not confirm this, mark it `Unconfirmed`.
- It is not a duplicate. Merge repeated instances into one issue and list every location.

Drop anything that fails these checks.

## Step 5 — Assign Severity

| Severity | Meaning |
| --- | --- |
| **Critical** | Exploitable now, or causes data loss or exposure (committed secrets, authentication bypass, injection) |
| **High** | Serious security, data, or functional flaw that must be fixed before deployment |
| **Medium** | Affects reliability or maintainability but is not immediately exploitable |
| **Low** | Style, readability, and minor improvements |

When torn between two levels, choose the lower one and explain why in the Problem field.

## Step 6 — Write the Fix

For every issue, give numbered steps. Add a corrected code solution whenever a code change is the fix. This is required for all Critical and High issues, and for Medium and Low issues when it helps.

Corrected code must be correct, not illustrative:

- Use the project's real language, framework, and versions.
- Reuse names, imports, types, and patterns from the surrounding code.
- Keep it minimal. Show only the lines that change, with enough context to place them.
- Do not invent libraries, methods, or config keys. If you are unsure an API exists in the project's version, say so.
- If the right fix depends on code or decisions you cannot see, say what is needed instead of guessing.

## Step 7 — Write the Report

Save to `reports/NN-feature-name-audit.md` using exactly this structure.

````markdown
# Audit Report — [Feature Name]

Audited folder: [SOURCE_FOLDER]
Date: [date]
Files reviewed: [count]

## Summary

| Severity | Count |
| --- | --- |
| Critical | [n] |
| High | [n] |
| Medium | [n] |
| Low | [n] |
| **Total** | [n] |

## Critical
[Issues, or "None found"]

## High
[Issues, or "None found"]

## Medium
[Issues, or "None found"]

## Low
[Issues, or "None found"]

## Top Priorities
1. [ID] — [one-line reason]
2. [ID] — [one-line reason]
3. [ID] — [one-line reason]
````

Use this block for every issue, numbered continuously (C-1, H-1, M-1, L-1):

````markdown
### [ID] — [Short title]

- **Category:** Security | Data leak | Functionality | Maintainability | Readability
- **Location:** `path/to/file.ext` lines [start]-[end]
- **Status:** Confirmed | Unconfirmed — [what could not be verified]
- **Problem:** [What is wrong and why it matters]
- **How to fix it:**
  1. [Step]
  2. [Step]
- **Solution:**

  Before:
  ```[lang]
  [current code]
  ```
  After:
  ```[lang]
  [corrected code]
  ```
````

Report rules:

- Order issues Critical → High → Medium → Low. Within a level, most dangerous first.
- Write "None found" under any empty severity.
- Keep "Top Priorities" to 3–5 items, ordered by risk.
- Keep each issue short. Do not repeat the same explanation across issues.

## Step 8 — Final Check and Reply

Before finishing, confirm:

- No file inside `SOURCE_FOLDER` was changed.
- The report exists in `reports/` with the correct name and number.
- No existing report was overwritten.
- Summary counts match the issues listed.
- No real secret values appear in the report.

Then reply with only:

```
Audit complete → reports/[file name]

[Total] issues: [n] Critical, [n] High, [n] Medium, [n] Low.
[n] marked Unconfirmed.
```

Stop there. Do not apply any fix. If the developer asks for fixes, treat that as a separate task.
