---
name: i18n-check
description: Find hardcoded user-facing text and missing translations across the project's supported languages. Checks UI, emails, errors, and formatting of dates, numbers, and currency. Reports only, never fixes.
---

Translation bugs are cheap to prevent and expensive to find later. A single hardcoded string means one language shows English forever. This skill finds them before users do.

Run it after building any feature that shows text to a user.

## What This Skill Does Not Do

It does not translate, edit translation files, or change code. Translating is a separate request. It reports what is missing.

## How to Invoke

```
/i18n-check [path]
```

If no path is given, check the files changed on the current branch (`git diff main`). If there are no changes, ask which area to check.

---

## Step 1 — Learn the Setup

Read the supported languages from `project-overview.md` or the project documentation. Find the translation library and where translation files live (for example `messages/en.json`). Find how the backend localizes emails and error messages. Match the project's existing pattern. Do not suggest a new library.

## Step 2 — Check

**Hardcoded text in the frontend**
- Visible text inside components that does not go through the translation function
- Text in attributes: `placeholder`, `title`, `alt`, `aria-label`, button and link labels
- Toast, dialog, validation, and empty-state messages
- Page titles and metadata

**Hardcoded text in the backend**
- Email subjects and bodies that are not localized
- Error messages shown to users instead of stable error codes the frontend can translate
- PDF text (invoices, reports) not localized

**Translation files**
- Keys that exist in one language and are missing in another
- Keys used in code but defined nowhere
- Keys defined but never used
- Values still in the source language in a target-language file (untranslated copies)
- Placeholder or variable mismatches between languages (for example `{count}` present in one language and missing in another)
- Plural and gender rules missing where the text needs them

**Formatting**
- Dates, times, numbers, and currency built by string concatenation instead of the locale formatting APIs
- Currency symbol hardcoded instead of coming from the currency code
- Dates shown without the correct timezone where the project requires one
- Text built by joining translated fragments, which breaks word order in other languages

**Layout risks**
- Fixed-width containers around text that can grow in other languages
- Text that is truncated with no way to read it

**Do not flag**
- Code identifiers, logs, test files, developer-only text, enum values never shown to users, and brand names.

## Step 3 — Report

Evidence only: a real file and line for every finding.

```
## i18n Check — [Area]

Languages: [list]

### Hardcoded text
| File:line | Text | Type |
| --- | --- | --- |

### Missing translations
| Key | Missing in |
| --- | --- |

### Unused or undefined keys
[List]

### Variable mismatches
[Key, language, what differs]

### Formatting problems
[File:line, what is wrong]

### Layout risks
[File:line, why]

### Summary
[X] hardcoded strings, [Y] missing translations, [Z] formatting problems.
```

Severity labels: **Important** for user-visible hardcoded text and missing keys, **Minor** for unused keys and layout risks.

If nothing is found, say: "No i18n issues found in [area]."

## Step 4 — Let the Developer Decide

Stop after the report. Do not fix or translate anything unless asked. If asked to translate, mark machine translations clearly so a native speaker can review them.

## The Standard

Every string a user can read exists in every supported language, and every date, number, and price is formatted for the user's locale. English text appearing in another language is a bug.
