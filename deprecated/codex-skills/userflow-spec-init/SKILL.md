---
name: userflow-spec-init
description: Use when a user wants to initialize docs//userflows/userflow-spec.md from the product brief and fill only sections 1 and 2. Checks for project-brief/project-brief.md first, creates the userflow directories and spec file from the bundled assets/userflow-spec.template.md, copies Product Name and Product Definition from the product brief, then guides the user through Userflow Goal using bundled references/userflow-spec.guidance.md.
---

# Userflow Spec Init

Use this skill to initialize the central userflow spec for a project.

Default required input file:

```text
project-brief/project-brief.md
```

Bundled template:

```text
assets/userflow-spec.template.md
```

Bundled guidance:

```text
references/userflow-spec.guidance.md
```

Bundled paths are relative to this skill directory. Do not assume the skill is
installed at a specific project path.

Default output file:

```text
docs//userflows/userflow-spec.md
```

The skill is complete when sections 1 and 2 are filled in
`docs//userflows/userflow-spec.md`.

Do not fill the Flow Registry during this skill unless the user explicitly
changes the task.

## Hard Stop Rule

Before doing anything else, check whether this file exists:

```text
project-brief/project-brief.md
```

If it does not exist, stop immediately and tell the user:

```text
I cannot initialize the userflow spec yet because project-brief/project-brief.md does not exist.
Create the product brief first, then I can initialize the userflow spec from it.
```

Do not create `userflow-spec.md` without the product brief.

## Required Reference Rule

Always read these files before drafting or writing the userflow spec:

```text
project-brief/project-brief.md
assets/userflow-spec.template.md
references/userflow-spec.guidance.md
```

Use the template for structure.

Use the guidance file for how each section should be filled.

## Existing File Behavior

If `docs//userflows/userflow-spec.md` already exists:

1. Read it.
2. Tell the user it already exists.
3. Inspect whether sections 1 and 2 are already filled.
4. If sections 1 and 2 are complete, do not overwrite them. Ask whether the
   user wants to revise them.
5. If sections 1 or 2 are incomplete, continue by filling only the missing or
   placeholder values.
6. Never overwrite a non-placeholder user-written value without explicitly
   confirming with the user.

Placeholder values include text wrapped in angle brackets, such as:

```text
<Product name>
<Which user journeys should this userflow set clarify, and what product decisions should it help the team make?>
```

## Directory Creation

When initialization is allowed, ensure these directories exist:

```text
docs//userflows
docs//userflows/flows
```

Then create or update:

```text
docs//userflows/userflow-spec.md
```

## Section 1 Filling Rule

Fill section 1 by copying from `project-brief/project-brief.md`.

Required source fields:

```text
## 1. Product Name
## 2. Product Definition
```

Copy the product name into:

```text
## 1. Product Brief
### Product Name
```

Copy the product definition into:

```text
## 1. Product Brief
### Product Definition
```

If either source field is missing or empty, stop and tell the user exactly which
field is missing from the product brief.

## Section 2 Collaboration Rule

Fill section 2 with the user.

Ask for:

```text
Userflow Goal
```

Briefly explain the field using `references/userflow-spec.guidance.md`.

Offer a recommendation when the product brief gives enough context.

Ask the user to confirm, revise, or provide the value.

Save the accepted value into the draft state.

## Recommended Defaults

Use recommendations only as suggestions. The user must still confirm or revise
them.

For a web chat product, derive a concise product-facing `Userflow Goal` from the
product brief. Example:

```text
Clarify the main journeys from first visit through authenticated chat use, including how guests sign up or log in and how signed-in users continue conversations.
```

## User Communication Style

Be clear about what the user and AI are doing together.

Good opening after the product brief exists:

```text
I will initialize docs//userflows/userflow-spec.md from the product brief.
Section 1 will be copied from project-brief/project-brief.md.
For section 2, I will ask for the Userflow Goal and use the userflow spec guidance to keep it consistent.
```

If the user seems unsure, give a recommendation and ask for confirmation:

```text
For this product, I recommend this Userflow Goal:
Clarify the main journeys from first visit through authenticated chat use, including how guests sign up or log in and how signed-in users continue conversations.
Should I use this?
```

## Drafting And Write Rules

Keep a working draft in memory while asking for the Userflow Goal.

Before writing, show the user the final section 1 and section 2 draft and ask
for approval.

Only write the file after approval.

When writing:

1. Start from `assets/userflow-spec.template.md`.
2. Replace section 1 and section 2 placeholders with accepted values.
3. Leave section 3 as the template placeholder Flow Registry.

## End Condition

The task is complete only when:

- `docs//userflows/userflow-spec.md` exists.
- Section 1 is filled from `project-brief/project-brief.md`.
- Section 2 has an accepted value for Userflow Goal.
- Section 3 remains the unfilled template Flow Registry.

After writing, summarize:

- product brief source path
- userflow spec output path
- the value used for Userflow Goal
- any fields left for future work
