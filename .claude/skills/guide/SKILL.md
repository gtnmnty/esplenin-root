---
name: guide
description: Explain step by step how to implement a feature or task in the project's own language and framework, in simple words, with the concepts needed, why they matter, and short code examples. Read-only. Never edits project files.
---

Knowing what to build is half the work. Knowing how to build it in your stack is the other half. This skill teaches the second half, in plain words, so you understand the code before it exists.

Run it after `/architect` has settled the plan and before you build.

## What This Skill Does Not Do

- It does not create, edit, or delete any project file. Code appears in the chat as examples only.
- It does not make design decisions. If the task still needs decisions, it sends you to `/architect`.
- It does not review or test code. That is for `/review` and `/test`.

## How to Invoke

```
/guide [task or feature]
```

Example: `/guide add an endpoint that cancels an order`. If the task is missing, ask for it. If the task could mean two different things, ask one short question before answering.

---

## Step 1 — Learn the Project

Before writing anything, read:

- `architecture.md` — stack, layers, folders, invariants
- `code-standards.md` — naming, error handling, validation, folder layout
- The dependency file (`pom.xml`, `package.json`, `requirements.txt`, etc.) — exact library versions
- Any existing code that is similar to the task, so the guide matches how the project already does things
- The plan from `/architect`, if one exists

Find out the language, the framework, and its version. Everything in the guide must fit that stack. Do not explain a Next.js pattern in a Spring Boot project.

## Step 2 — Check the Task Is Ready

Stop and say so if:

- **Decisions are missing.** Behavior, limits, or rules are not defined. Say: "This needs decisions first. Run `/architect`." List the open decisions.
- **It breaks an invariant.** If the task, done the obvious way, would violate a rule in `architecture.md`, say which rule and teach the allowed way.
- **It is too big.** If it needs more than 8 steps, split it into parts and guide only the first part. Name the later parts.

## Step 3 — Write the Guide

Use this exact shape.

````
## Guide — [Task]

### Goal
[One sentence: what will work when you finish.]

### Concepts you need
**[Concept name]** — [What it is, in one or two plain sentences.]
Why you need it here: [One sentence.]

(Repeat for each concept. Only list concepts this task really needs.)

### Steps

**Step 1 — [Short action]**
What to do: [Plain instruction.]
Why: [One or two sentences.]
```[language]
[Short example]
```

(Repeat. Between 3 and 8 steps.)

### Common mistakes
- [Mistake] — [How to avoid it]

### How to check it works
[Two or three concrete checks: a request to send, a result to expect, a test to run.]

### Next
[The next skill or step: usually `/test`, then `/review`.]
````

## Writing Rules

**Words**
- Use short sentences and everyday words.
- Explain a technical term the first time it appears, in one plain sentence.
- Say what a thing does before what it is called.
- No filler, no praise, no long introductions.

**Code**
- Keep each example short, about 15 lines at most. Show only the part that matters.
- Use the project's real names (classes, folders, packages, error codes) when they exist. If they do not exist yet, use clear placeholder names and say they are placeholders.
- Follow `code-standards.md` exactly: layering, validation, error format, naming.
- Use only real library methods that exist in the version the project uses. If you are not sure a method exists in that version, say so instead of guessing.
- Never include real secrets, keys, or passwords. Use placeholders like `YOUR_API_KEY`.
- Never show a shortcut that breaks security (for example building SQL from strings, or trusting a price sent by the browser).
- Add a comment only where the code is not obvious.

**Accuracy**
- Only teach what the project's stack supports. Do not suggest a new library when the project already has one for the job.
- If something depends on a choice you cannot see in the project, say what it depends on.
- Separate facts from suggestions. Do not present a guess as fact.

**Size**
- A guide is a page, not a book. If it is getting long, cut concepts that are not needed for this task.

## Step 4 — Let the Developer Continue

After the guide, stop. Do not start building. Do not edit any file. Offer nothing more than the Next line.

If the developer asks a follow-up ("explain step 3 again", "show it for the update case"), answer only that, in the same style.

## The Standard

A beginner who reads the guide should know what to build, why each piece exists, and how to tell it works. If they would need to ask "why?" at any step, the guide is not finished.
