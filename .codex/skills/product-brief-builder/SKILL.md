---
name: product-brief-builder
description: Use when a user wants to create, fill, draft, revise, or update a root product brief for a project. Guides the user interactively through a concise product brief template with flexible unordered input, create/update mode detection, section-by-section confirmation, file path confirmation, and final approval before writing project-brief/project-brief.md.
---

# Product Brief Builder

Use this skill to help a user create or update a root product brief. The brief
is a concise central product document, not a PRD, feature spec, technical
architecture doc, or roadmap.

Default project file:

```text
project-brief/project-brief.md
```

Default template asset:

```text
assets/product-brief.template.md
```

## Core Rules

- Do not rush to write the file.
- Ask only 1-3 focused questions at a time.
- Accept messy, unordered user input.
- Use flexible slot filling instead of forcing the user through sections in order.
- Keep the brief concise and stable enough to live as a root product document.
- Clearly separate known facts, assumptions, and missing information.
- Confirm mode, path, draft content, and final write approval before editing files.
- Do not place implementation details in the product brief.

## Product Brief Sections

The final brief should contain these sections:

```text
1. Product Definition
2. Product Purpose
3. Intended Users
4. Core Experience
5. Current Capabilities
6. Future Opportunities
7. Product Constraints
```

## Mode Detection

Detect the intended mode from the user's request and file state.

Use create mode when:

- the product brief file does not exist
- the user says they want a new product brief
- the user wants to start from a blank product idea
- the user asks to fill the template from scratch

Use update mode when:

- the product brief file exists
- the user says update, revise, change, add, remove, or fix the brief
- the user describes a product-level change to an existing project

Always confirm ambiguous mode:

```text
I found an existing product brief at project-brief/project-brief.md.
Should I update that file, or start a new draft?
```

## Slot-Filling Method

Treat each section as a slot. When the user says anything useful, map it into
the relevant slot even if it arrives out of order.

Working state:

```text
Known:
- Product Definition: ...
- Product Purpose: ...

Assumptions:
- ...

Still missing or weak:
- ...
```

After each meaningful user answer:

1. Extract usable details.
2. Update the working state.
3. Briefly tell the user what is now understood.
4. Ask the smallest useful next question.

Do not ask about a section that already has enough information unless it is
unclear, contradictory, or too implementation-specific.

## Minimum Information Needed

Before drafting, aim for:

- Product Definition: clear
- Product Purpose: clear enough
- Intended Users: clear enough
- Core Experience: clear enough
- Current Capabilities: at least 3 product-level bullets
- Future Opportunities: can be short or "None identified yet"
- Product Constraints: at least 2-3 useful constraints

Do not block forever seeking perfection. If a section is weak, mark an
assumption and ask the user to confirm it.

## Create Mode Workflow

1. Explain the process briefly.
2. Confirm the target file path.
3. Ask for the raw product idea.
4. Use slot filling to map the idea into the seven sections.
5. Ask follow-up questions for missing or weak sections.
6. Present a concise draft.
7. Ask the user to revise or approve the draft.
8. After approval, ask final write confirmation.
9. Write the file only after final confirmation.
10. Summarize the created file and any open follow-up docs.

Good opening:

```text
I will help fill the root product brief slowly. I will collect the idea, map it
into the seven sections, show a draft, and only write the file after you confirm.
I recommend docs//01-project-brief.md. Should I use that path?
```

## Update Mode Workflow

1. Read the existing product brief.
2. Ask what changed.
3. Classify whether the change belongs in the product brief.
4. If it belongs, identify affected sections.
5. Ask follow-up questions if the change is unclear.
6. Show a targeted before/after or full revised draft.
7. Ask for approval.
8. Confirm write path.
9. Write only after final confirmation.
10. Summarize the update.

Changes that usually belong in the product brief:

- product definition changes
- product purpose changes
- intended users change
- core experience changes
- current product capabilities change
- future product opportunities change
- product constraints change

Changes that usually do not belong in the product brief:

- endpoint changes
- database field changes
- framework/library changes
- folder structure changes
- deployment commands
- detailed page flows
- detailed milestones
- feature implementation details

If a change does not belong, redirect it:

```text
That sounds implementation-specific, so it probably should not change the root
product brief. It belongs in the backend project docs or a feature spec instead.
```

## Question Strategy

Prefer broad questions first, then targeted follow-ups.

Useful create-mode questions:

- What is the product in plain language?
- Who is it designed for?
- What problem should it solve?
- What should the user be able to do in the core experience?
- What capabilities are part of the product shape now?
- What might the product expand toward later?
- What constraints should future design and engineering decisions respect?

Useful update-mode questions:

- What changed about the product?
- Is this a product-level change, or an implementation detail?
- Which users or flows are affected?
- Does this change current capabilities or future opportunities?
- Should this replace existing wording or add a new bullet?

## Drafting Rules

Use direct, stable language.

Avoid:

- "first audience"
- "rough"
- "maybe" unless explicitly describing future opportunities
- endpoint names
- database tables
- framework names
- excessive roadmap detail

Prefer:

- one or two short paragraphs for definition, purpose, users, and experience
- concise bullets for capabilities, opportunities, and constraints
- product-level language that frontend, backend, and AI-server developers can
  all understand

## File Writing Rules

Before any edit, show:

```text
Target file:
project-brief/project-brief.md

Draft:
<markdown draft>

Do you want me to write this file?
```

Only write the file after clear confirmation.

Use the repo template if present:

```text
docs/templates/product-brief-template.md
```

If both `docs/templates/product-brief-template.md` and the skill asset exist,
prefer the repo template because it is project-specific.

