---
name: wireframe-relation-register
description: Use when a user wants to add rows to the Frame Relation Map table in docs/central/wireframes/wireframe-spec.md, either manually or from AI suggestions. Shows the Frame Registry first, asks whether to add manually or get suggestions, inspects existing relation rows and frame markdown files when useful, validates that From Frame and To Frame exist and that the relation makes visual/product sense, then writes approved rows using bundled references/wireframe-spec.guidance.md.
---

# Wireframe Relation Register

Use this skill to add one or more rows to the `## 4. Frame Relation Map` table
in the central wireframe spec.

Default target file:

```text
docs/central/wireframes/wireframe-spec.md
```

Bundled spec guidance:

```text
references/wireframe-spec.guidance.md
```

Bundled frame guidance:

```text
references/wireframe-frame.guidance.md
```

Bundled paths are relative to this skill directory. Do not assume the skill is
installed at a specific project path.

## Hard Stop Rule

Before doing anything else, check whether this file exists:

```text
docs/central/wireframes/wireframe-spec.md
```

If it does not exist, stop immediately and tell the user:

```text
I cannot register frame relations yet because docs/central/wireframes/wireframe-spec.md does not exist.
Initialize the wireframe spec and register frames first, then I can add relation rows.
```

Do not create `wireframe-spec.md` in this skill.

## Required Reference Rule

Always read these files before adding relation rows:

```text
docs/central/wireframes/wireframe-spec.md
references/wireframe-spec.guidance.md
```

Use `references/wireframe-spec.guidance.md` as the authority for how to fill
the Frame Relation Map columns.

Read `references/wireframe-frame.guidance.md` only when inspecting frame
markdown files or reasoning about regions/components.

## Scope Rule

This skill only adds rows to the Frame Relation Map table.

Do not:

- create new frame registry rows
- create or rewrite frame markdown files
- draw in Figma
- change frame statuses
- edit sections 1, 2, or 3 unless the user explicitly asks for a correction

## Start Flow

Read the `## 3. Frame Registry` table and show it to the user first.

Use a compact table:

```text
Frame ID | Frame Name | Frame Category | Base Frame | Frame Status | Frame Spec File
```

Then ask whether the user wants:

```text
manual add
```

or:

```text
suggestions
```

Do not continue until the user chooses one mode.

## Relation Map Columns

Every new relation row must use these columns:

```text
From Frame | Trigger / Event | To Frame | What Changes
```

Write `What Changes` as a present-tense visual/product statement.

Example:

```text
Signup modal opens over the landing frame.
```

## Validation Rules

Validate every proposed row before asking for write approval.

Required checks:

- `From Frame` exists in the Frame Registry.
- `To Frame` exists in the Frame Registry.
- `From Frame` and `To Frame` are not the same unless the row clearly describes
  a meaningful state change.
- `Trigger / Event` is product-facing, not implementation detail.
- `What Changes` describes a visible frame change in present tense.
- The row is not a duplicate of an existing relation row.
- The relation is logically consistent with frame categories and base frames.

Category logic examples:

- A `Base` to `Overlay` relation usually says an overlay opens.
- An `Overlay` to its base frame usually says the overlay closes.
- A `Base` to `State` relation usually says the frame changes state.
- A successful auth relation usually moves from a public/auth frame to an
  authenticated base frame.

If a row fails validation, explain the issue and ask the user how to revise it.

## Manual Add Mode

Help the user add one relation row interactively.

Ask one field at a time:

1. From Frame
2. Trigger / Event
3. To Frame
4. What Changes

For `From Frame` and `To Frame`, let the user provide either a `WF-XXX` ID or a
frame name. Resolve names to IDs using the Frame Registry.

If the user is unsure, suggest likely frames from the registry.

After all fields are collected:

1. Validate the row.
2. Show the exact row.
3. Ask for approval.
4. Write only after approval.

After writing one row, ask whether the user wants to add another relation row.

## Suggestion Mode

Use suggestion mode to find likely missing relation rows.

Inspect:

1. Frame Registry rows.
2. Existing Frame Relation Map rows.
3. Frame categories and base frame relationships.
4. Frame markdown files when useful and available from `Frame Spec File`.

Read frame markdown files only when they help identify triggers, states,
overlays, regions, or components. Do not read nonexistent or `TBD` frame files.

### Suggestion Heuristics

Suggest likely missing rows when:

- An `Overlay` frame exists but no row opens it from its base frame.
- An `Overlay` frame exists but no row closes it back to its base frame.
- A `State` frame exists but no row changes from its base frame to that state.
- A public/auth frame and authenticated base frame exist but no auth-success
  relation connects them.
- A `Detail` frame exists but no row shows how the user opens that detail.
- Frame markdown mentions visible actions that appear to open, close, replace,
  or navigate to another frame.

Do not invent speculative product flows that are not supported by the registry,
frame names, frame categories, base frame values, or frame markdown content.

### Suggestion Format

Show suggestions as a table:

```text
Suggestion ID | From Frame | Trigger / Event | To Frame | What Changes | Reason
```

Keep reasons short and concrete.

Example reason:

```text
WF-002 is an Overlay over WF-001, but no row opens it yet.
```

Ask which suggestions the user wants to add.

The user may choose:

- all
- none
- specific suggestion IDs
- revised versions of suggestions

Before writing, validate all selected suggestions again and show the exact rows
that will be added.

Write only after approval.

## Existing Placeholder Row Rule

If the Frame Relation Map table only contains the template placeholder row,
replace that placeholder row with the first real relation row.

A placeholder row usually contains values like:

```text
<User action or system event>
<What visually changes?>
```

If the table already contains real rows, append approved new rows after the last
existing relation row.

## Write Rules

When writing:

1. Preserve all existing content.
2. Modify only the Frame Relation Map table row area.
3. Replace the placeholder row only when it is the only relation row.
4. Otherwise append approved rows after the last existing relation row.
5. Do not add duplicate rows.

## Terminal Conditions

The skill ends when one of these is true:

- Approved manual row or rows have been written.
- Approved suggestion row or rows have been written.
- The user chooses not to add any rows.
- No valid suggestions exist and the user does not want to add manually.
- Required files or valid frames are missing, so the skill cannot proceed.

After writing, summarize:

- target file path
- rows added
- rows skipped or rejected, if any
- any remaining likely missing relations
