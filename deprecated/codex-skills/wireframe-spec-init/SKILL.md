---
name: wireframe-spec-init
description: Use when a user wants to initialize docs//wireframes/wireframe-spec.md from the product brief and fill only sections 1 and 2. Checks for project-brief/project-brief.md first, creates the wireframe directories and spec file from the bundled assets/wireframe-spec.template.md, copies Product Name and Product Definition from the product brief, then guides the user step by step through Platform, Wireframe Goal, Fidelity, and Drawing Tool using the bundled references/wireframe-spec.guidance.md.
---

# Wireframe Spec Init

Use this skill to initialize the central wireframe spec for a project.

Default required input file:

```text
project-brief/project-brief.md
```

Bundled template:

```text
assets/wireframe-spec.template.md
```

Bundled guidance:

```text
references/wireframe-spec.guidance.md
```

Bundled paths are relative to this skill directory. Do not assume the skill is
installed at a specific project path.

Default output file:

```text
docs//wireframes/wireframe-spec.md
```

The skill is complete when sections 1 and 2 are filled in
`docs//wireframes/wireframe-spec.md`.

Do not fill the Frame Registry or Frame Relation Map during this skill unless
the user explicitly changes the task.

## Hard Stop Rule

Before doing anything else, check whether this file exists:

```text
project-brief/project-brief.md
```

If it does not exist, stop immediately and tell the user:

```text
I cannot initialize the wireframe spec yet because project-brief/project-brief.md does not exist.
Create the product brief first, then I can initialize the wireframe spec from it.
```

Do not create `wireframe-spec.md` without the product brief.

## Required Reference Rule

Always read these files before drafting or writing the wireframe spec:

```text
project-brief/project-brief.md
assets/wireframe-spec.template.md
references/wireframe-spec.guidance.md
```

Use the template for structure.

Use the guidance file for how each section should be filled.

## Existing File Behavior

If `docs//wireframes/wireframe-spec.md` already exists:

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
<What should be easier to decide after this wireframe set exists?>
```

## Directory Creation

When initialization is allowed, ensure these directories exist:

```text
docs//wireframes
docs//wireframes/frames
```

Then create or update:

```text
docs//wireframes/wireframe-spec.md
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

Fill section 2 with the user step by step.

Ask for one field at a time, in this order:

1. Platform
2. Wireframe Goal
3. Fidelity
4. Drawing Tool

For each field:

1. Briefly explain what the field means using
   `references/wireframe-spec.guidance.md`.
2. Offer a recommendation when the product brief gives enough context.
3. Ask the user to confirm, revise, or provide the value.
4. Save the accepted value into the draft state.

Do not ask all four questions at once.

## Recommended Defaults

Use recommendations only as suggestions. The user must still confirm or revise
them.

For a web chat product:

```text
Platform: Responsive web
Fidelity: Low fidelity
Drawing Tool: TBD
```

For `Wireframe Goal`, derive a concise product-facing goal from the product
brief. Example:

```text
Clarify the first user-facing chat flow, unauthenticated auth entry, and authenticated chat shell before detailed frame specs are created.
```

## User Communication Style

Be clear about what the user and AI are doing together.

Good opening after the product brief exists:

```text
I will initialize docs//wireframes/wireframe-spec.md from the product brief.
Section 1 will be copied from project-brief/project-brief.md.
For section 2, I will ask you one field at a time and use the wireframe spec guidance to keep the answers consistent.
```

If the user seems unsure, give a recommendation and ask for confirmation:

```text
For this product, I recommend Responsive web because the product brief describes a ChatGPT-style web application.
Should I use Responsive web for Platform?
```

## Drafting And Write Rules

Keep a working draft in memory while asking section 2 questions.

Before writing, show the user the final section 1 and section 2 draft and ask
for approval.

Only write the file after approval.

When writing:

1. Start from `assets/wireframe-spec.template.md`.
2. Replace section 1 and section 2 placeholders with accepted values.
3. Leave section 3 and section 4 as template placeholders.

## End Condition

The task is complete only when:

- `docs//wireframes/wireframe-spec.md` exists.
- Section 1 is filled from `project-brief/project-brief.md`.
- Section 2 has accepted values for Platform, Wireframe Goal, Fidelity, and
  Drawing Tool.
- Section 3 and section 4 remain unfilled template sections.

After writing, summarize:

- product brief source path
- wireframe spec output path
- the values used for section 2
- any fields left for future work
