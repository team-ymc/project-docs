---
name: wireframe-frame-register
description: Use when a user wants to add a new frame row to the Frame Registry table in docs//wireframes/wireframe-spec.md. Guides the user one column at a time using bundled references/wireframe-spec.guidance.md, assigns the next WF-XXX Frame ID, sets Frame Status to Not Started, and sets Frame Spec File to TBD.
---

# Wireframe Frame Register

Use this skill to add one new row to the `## 3. Frame Registry` table in the
central wireframe spec.

Default target file:

```text
docs//wireframes/wireframe-spec.md
```

Bundled guidance:

```text
references/wireframe-spec.guidance.md
```

Bundled paths are relative to this skill directory. Do not assume the skill is
installed at a specific project path.

The skill is complete when exactly one new frame row has been added to the
Frame Registry table.

## Hard Stop Rule

Before doing anything else, check whether this file exists:

```text
docs//wireframes/wireframe-spec.md
```

If it does not exist, stop immediately and tell the user:

```text
I cannot register a new wireframe frame yet because docs//wireframes/wireframe-spec.md does not exist.
Initialize the wireframe spec first, then I can add a new frame row.
```

Do not create `wireframe-spec.md` in this skill.

## Required Reference Rule

Always read these files before adding a frame row:

```text
docs//wireframes/wireframe-spec.md
references/wireframe-spec.guidance.md
```

Use the guidance file as the authority for how to fill Frame Registry columns.

## Scope Rule

This skill only adds a new row to the Frame Registry table.

Do not:

- create the frame spec file
- fill the Frame Relation Map
- edit sections 1 or 2
- create drawings
- change existing frame rows unless the user explicitly asks for a correction

## New Row Values

The new row must use these columns:

```text
Frame ID | Frame Name | Frame Category | Base Frame | Frame Status | Frame Spec File
```

The user helps fill:

```text
Frame Name
Frame Category
Base Frame
```

The skill fills automatically:

```text
Frame ID: next available WF-XXX ID
Frame Status: Not Started
Frame Spec File: TBD
```

## Frame ID Rule

Find all existing `WF-XXX` IDs in the Frame Registry table.

Choose the next highest number.

Examples:

```text
Existing: WF-001, WF-002
New: WF-003
```

```text
Existing: WF-001, WF-003
New: WF-004
```

Do not reuse deprecated, deleted, or missing IDs.

## Existing Placeholder Row Rule

If the Frame Registry table only contains the template placeholder row, replace
that placeholder row with the new real row.

A placeholder row usually contains values like:

```text
<Frame name>
<Base / Overlay / State / Detail>
<none or WF-XXX>
```

If the table already contains real frame rows, append the new row after the last
existing frame row.

## User Collaboration Flow

Explain the process before asking questions:

```text
I will add one new row to the Frame Registry.
I will determine the next Frame ID automatically.
I will ask you for Frame Name, Frame Category, and Base Frame.
Frame Status will be Not Started, and Frame Spec File will be TBD.
```

Ask for one value at a time.

### Step 1: Frame Name

Ask what the frame should be called.

The name should be short and human-readable.

If the user gives a vague name, suggest a clearer one and ask for confirmation.

### Step 2: Frame Category

Ask which category applies.

Allowed values:

```text
Base
Overlay
State
Detail
```

Briefly explain each option from `references/wireframe-spec.guidance.md` if the
user is unsure.

### Step 3: Base Frame

Ask for the base frame based on the chosen category.

Rules:

```text
Base -> none
Overlay -> WF-XXX frame it appears over
State -> WF-XXX frame it is a state of
Detail -> WF-XXX frame that contains the focused region
```

If the user does not know the exact frame ID, inspect the existing Frame
Registry rows and recommend likely options.

If no valid base frame exists yet for an Overlay, State, or Detail frame, tell
the user that a related Base frame should usually be registered first.

## Draft And Approval Rule

Before editing the file, show the exact row to be added:

```text
| WF-XXX | <Frame Name> | <Frame Category> | <Base Frame> | Not Started | TBD |
```

Ask for approval before writing.

Only edit `docs//wireframes/wireframe-spec.md` after approval.

## Write Rule

When writing:

1. Preserve all existing content.
2. Modify only the Frame Registry table row area.
3. Replace the placeholder row if it is the only row.
4. Otherwise append the new row after the last existing frame row.

## End Condition

The task is complete only when:

- one new row exists in the Frame Registry table
- `Frame ID` uses the next available `WF-XXX`
- `Frame Name` uses the accepted user value
- `Frame Category` is one of `Base`, `Overlay`, `State`, `Detail`
- `Base Frame` follows the category rule
- `Frame Status` is `Not Started`
- `Frame Spec File` is `TBD`

After writing, summarize the added row and the target file path.
