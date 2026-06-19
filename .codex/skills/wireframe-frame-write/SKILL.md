---
name: wireframe-frame-write
description: Use when a user wants to write, create, continue, or revise an individual docs//wireframes/frames/WF-XXX-frame-name.md wireframe frame spec from a row in docs//wireframes/wireframe-spec.md. Shows the Frame Registry, lets the user choose a frame, creates the frame file from bundled assets/wireframe-frame.template.md when status is Not Started, updates the registry status/path to In Progress, and interactively fills Visual Purpose, Layout Regions, Component Inventory, and a Markdown Sketch using bundled references/wireframe-frame.guidance.md.
---

# Wireframe Frame Write

Use this skill to create or continue one individual wireframe frame spec file.

Default registry file:

```text
docs//wireframes/wireframe-spec.md
```

Default frame directory:

```text
docs//wireframes/frames
```

Bundled frame template:

```text
assets/wireframe-frame.template.md
```

Bundled frame guidance:

```text
references/wireframe-frame.guidance.md
```

Bundled spec guidance:

```text
references/wireframe-spec.guidance.md
```

Bundled paths are relative to this skill directory. Do not assume the skill is
installed at a specific project path.

The skill is complete when the selected frame file exists and sections 1-4 are
filled enough for the user to continue reviewing, translating into user flows,
or drawing the frame.

## Hard Stop Rule

Before doing anything else, check whether this file exists:

```text
docs//wireframes/wireframe-spec.md
```

If it does not exist, stop immediately and tell the user:

```text
I cannot author a frame file yet because docs//wireframes/wireframe-spec.md does not exist.
Initialize the wireframe spec and register a frame first, then I can write the frame file.
```

Do not create `wireframe-spec.md` in this skill.

## Required Reference Rule

Always read these files before creating or editing a frame file:

```text
docs//wireframes/wireframe-spec.md
assets/wireframe-frame.template.md
references/wireframe-frame.guidance.md
references/wireframe-spec.guidance.md
```

Use `references/wireframe-frame.guidance.md` as the authority for filling the
frame title, visual purpose, layout regions, component inventory, and current
markdown preview.

Use `references/wireframe-spec.guidance.md` only for registry status/path rules.

## Frame Selection Flow

Read the `## 3. Frame Registry` table from:

```text
docs//wireframes/wireframe-spec.md
```

Show the registry rows to the user in a compact table with:

```text
Frame ID | Frame Name | Frame Category | Base Frame | Frame Status | Frame Spec File
```

Ask which frame the user wants to handle.

If there is only one real frame row, recommend it but still ask for confirmation.

Do not continue until the user selects one frame.

## Status Behavior

### Not Started

If the selected frame status is `Not Started`:

1. Determine the frame file path.
2. Create the frame file from `assets/wireframe-frame.template.md`.
3. Replace the frame title with `# Frame: WF-XXX <Frame Name>`.
4. Update the selected registry row:
   - `Frame Status` becomes `In Progress`
   - `Frame Spec File` becomes the created file path
5. Start by helping the user fill `## 1. Visual Purpose`.

### In Progress

If the selected frame status is `In Progress`:

1. Read the frame file from the `Frame Spec File` path.
2. Show the current markdown version to the user.
3. Ask what the user wants to work on:
   - Visual Purpose
   - Layout Regions
   - Component Inventory
   - Markdown Sketch
   - general cleanup

If the `Frame Spec File` is `TBD` while status is `In Progress`, generate the
expected path, create the file if needed, and update the registry path after
confirming with the user.

### Finished

If the selected frame status is `Finished`, do not edit immediately.

Show the current frame file and ask whether the user wants to revise it. If the
user approves revision, change the registry status to `In Progress` before
editing.

### Deprecated

If the selected frame status is `Deprecated`, stop unless the user explicitly
asks to revive or inspect it.

## Frame File Path Rule

If `Frame Spec File` already contains a real path, use that path.

If `Frame Spec File` is `TBD`, generate the path from the selected frame row:

```text
docs//wireframes/frames/WF-XXX-frame-name.md
```

Rules:

- Start the filename with the exact `Frame ID`.
- Convert the frame name to lowercase kebab case.
- Remove punctuation that is not needed for meaning.
- Keep the path stable after creation.

## Base Frame Context Rule

If the selected row has a `Base Frame` value other than `none`:

1. Find that base frame row in the Frame Registry.
2. If its `Frame Spec File` path exists, read it.
3. Briefly summarize the base frame's regions/components before drafting.
4. Ask what is different in the selected frame.

Use this to help states, overlays, and detail frames stay consistent with their
base frame.

If the base frame file does not exist yet, continue without it and mention that
base-frame comparison is unavailable.

## Interactive Filling Flow

Work section by section. Ask focused questions and update the frame file after
the user accepts each meaningful draft.

Do not ask the user to fill large tables alone. Help propose rows from the
product context, frame name, base frame context, and user answers.

### 1. Visual Purpose

Ask what this frame should make clear at a glance.

If the user is unsure, recommend a concise visual purpose based on:

- frame name
- frame category
- base frame
- product context visible in the wireframe spec

Show the proposed sentence and ask for approval before writing it.

### 2. Layout Regions

Follow `references/wireframe-frame.guidance.md` exactly.

Build the table interactively:

1. Start with `R0 | Frame Root | None`.
2. Ask for the large visual areas in the frame.
3. Propose region rows with:
   - Region ID
   - Region Name
   - Parent Region
   - Placement Notes
   - Region Purpose
4. Show the proposed table.
5. Ask the user what is missing, wrong, or unnecessary.
6. Write only after the user accepts the table.

When a base frame file exists, ask whether the selected frame reuses, removes,
adds, or changes regions from the base frame.

### 3. Component Inventory

Follow `references/wireframe-frame.guidance.md` exactly.

Build the table interactively:

1. Walk through each region.
2. Ask what meaningful visible components belong in that region.
3. Propose component rows with:
   - Component ID
   - Component Name
   - Region ID
   - Component Type
   - Component Purpose
4. Show the proposed table.
5. Ask the user what is missing, wrong, or unnecessary.
6. Write only after the user accepts the table.

Do not list frontend implementation components. Use product/UI components only.

### 4. Markdown Sketch

After the layout regions and component inventory are accepted, create a simple
text sketch of the frame.

Use the accepted regions and components as the source of truth. The sketch
should show approximate visual placement, hierarchy, and visible labels in a
fenced `text` code block.

Keep the sketch intentionally rough. Use it to help the user see the wireframe
structure, not to specify exact spacing, dimensions, or visual style.

Do not duplicate the markdown spec in this section. Do not let the sketch become
the source of truth. The real source of truth is the filled sections above it.

Show the proposed sketch and ask for approval before writing it.

## Write Rules

Before editing files, show the draft change and ask for approval.

When writing:

1. Preserve existing user-written content unless the user approved replacing it.
2. Modify only the selected frame file and its selected registry row.
3. Keep table column order exactly as defined by the bundled frame template.
4. Keep section headings exactly as defined by the bundled frame template.
5. Use `apply_patch` or another precise file edit method.

## End Condition

The task is complete when:

- the selected frame file exists
- the registry row points to the frame file
- the registry status is `In Progress`, unless the user explicitly keeps another status
- `## 1. Visual Purpose` is filled
- `## 2. Layout Regions` has accepted rows
- `## 3. Component Inventory` has accepted rows
- `## 4. Markdown Sketch` shows an accepted rough text sketch of the frame

After writing, summarize:

- selected frame
- frame file path
- registry status/path update
- filled sections
- any open questions for future frame refinement
