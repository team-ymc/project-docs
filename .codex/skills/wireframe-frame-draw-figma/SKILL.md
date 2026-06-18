---
name: wireframe-frame-draw-figma
description: Use when a user wants to draw, create, update, or sync a Figma wireframe frame from an existing docs/central/wireframes/frames/WF-XXX-frame-name.md markdown frame spec. Always asks for and inspects the Figma file link first, shows the Frame Registry, only draws frames with In Progress status and sufficiently filled frame markdown, creates missing Figma frames or updates existing ones, syncs interactive Figma changes back to the markdown frame spec, and marks the registry frame status as Finished after the Figma frame and markdown spec are aligned.
---

# Wireframe Frame Draw Figma

Use this skill to draw or sync one wireframe frame in Figma from an existing
markdown frame spec.

Default registry file:

```text
docs/central/wireframes/wireframe-spec.md
```

Default frame directory:

```text
docs/central/wireframes/frames
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

## Figma Tool Rule

This skill requires Figma access.

Before any Figma write action, follow the active Figma tool prerequisites in
the current environment. If Figma write tools are unavailable or the file cannot
be accessed, stop and tell the user what is missing.

Do not pretend a Figma frame was created or updated unless the Figma operation
actually succeeded.

## First Step Rule

Always ask for the Figma file link first unless the user already provided it.

After receiving the link:

1. Access the Figma file.
2. Inspect the current file situation before selecting a frame.
3. Identify pages, existing wireframe frames, and any frames named with `WF-XXX`.
4. Briefly summarize what exists in the Figma file.

Do not draw before this inspection is complete.

## Hard Stop Rule

After checking the Figma file, check whether this file exists:

```text
docs/central/wireframes/wireframe-spec.md
```

If it does not exist, stop immediately and tell the user:

```text
I cannot draw a wireframe frame yet because docs/central/wireframes/wireframe-spec.md does not exist.
Initialize the wireframe spec and register/write a frame first, then I can draw it in Figma.
```

Do not create `wireframe-spec.md` in this skill.

## Required Reference Rule

Always read these files before drawing or syncing:

```text
docs/central/wireframes/wireframe-spec.md
assets/wireframe-frame.template.md
references/wireframe-frame.guidance.md
references/wireframe-spec.guidance.md
```

After the user chooses a frame, also read the selected frame markdown file from
the `Frame Spec File` path in the registry.

## Frame Selection Flow

Read the `## 3. Frame Registry` table from:

```text
docs/central/wireframes/wireframe-spec.md
```

Show the registry rows to the user in a compact table with:

```text
Frame ID | Frame Name | Frame Category | Base Frame | Frame Status | Frame Spec File
```

Ask which frame the user wants to draw.

Do not continue until the user selects one frame.

## Draw Eligibility Rule

Only draw a frame when the selected registry row has:

```text
Frame Status: In Progress
Frame Spec File: real markdown file path, not TBD
```

If the status is `Not Started`, stop and tell the user:

```text
This frame is Not Started, so I cannot draw it yet. Use the frame writing skill to create and fill the frame markdown first.
```

If the status is `Finished`, inspect the existing Figma frame and markdown, then
ask whether the user wants to resync or revise it.

If the status is `Deprecated`, stop unless the user explicitly asks to inspect
or revive it.

## Markdown Readiness Check

Even if the registry status is `In Progress`, inspect the frame markdown before
drawing.

The frame markdown is ready enough to draw only when:

- the title uses `# Frame: WF-XXX <Frame Name>`
- `## 1. Visual Purpose` has non-placeholder content
- `## 2. Layout Regions` has `R0` and at least one non-placeholder region row
- `## 3. Component Inventory` has meaningful component rows, or the frame
  clearly does not need visible components beyond regions
- placeholder values such as `<name>`, `<purpose>`, or `<type>` are not left in
  rows that should be drawn

If the markdown is not ready enough, stop and tell the user what is missing.
Recommend using `wireframe-frame-write` to finish the markdown first.

## Draw Confirmation Rule

After Figma inspection, frame selection, status check, and markdown readiness
check, summarize:

- selected frame
- frame markdown path
- whether a matching Figma frame already exists
- what will be created or updated

Then ask:

```text
Should I draw/sync this frame in Figma now?
```

Do not draw until the user confirms.

## Figma Frame Matching Rule

Use this Figma frame name format:

```text
WF-XXX <Frame Name>
```

When checking the Figma file:

- If no matching Figma frame exists, create a new Figma frame.
- If one matching Figma frame exists, update that frame to match the markdown.
- If multiple possible matches exist, ask the user which one to sync.

Prefer a page named `Wireframes` if it exists. If no suitable page exists, ask
before creating one.

## Drawing Rule

Draw a clear low-fidelity wireframe from the markdown frame spec.

Use the markdown as the source of truth:

- Visual Purpose informs the main emphasis.
- Layout Regions define major containers.
- Component Inventory defines visible UI elements inside regions.
- Placement Notes guide relative placement.
- Region Purpose and Component Purpose guide labels and grouping.

Keep the drawing wireframe-level:

- use simple boxes, labels, and neutral styling
- avoid production visual design
- avoid decorative images, gradients, and final branding unless the markdown
  explicitly requires them
- make regions and components readable
- preserve frame ID/name in the Figma frame title

## Existing Figma Frame Sync Rule

If the Figma frame already exists, update it to match the markdown.

Before updating, inspect whether there are differences between:

- current markdown frame spec
- current Figma frame

If there is nothing meaningful to sync, tell the user that the Figma frame
already appears aligned with the markdown and ask whether to mark it `Finished`.

Do not delete complex existing work without explaining the change and getting
approval.

## Interactive Figma Fix Rule

The user may ask to adjust the Figma frame during this skill session.

If the user requests a visual change:

1. Apply the Figma change.
2. Update the corresponding markdown frame spec so sections 1-4 still describe
   the Figma frame.
3. Show the markdown change summary.
4. Keep Figma and markdown aligned before finishing.

Never leave accepted Figma-only changes unsynced from the markdown.

## Markdown Sketch Sync Rule

Keep `## 4. Markdown Sketch` in the frame markdown synchronized with the drawn
wireframe structure.

The sketch is a user review aid. The real source of truth is the filled
sections above it.

## Registry Finish Rule

After the Figma frame is drawn or confirmed aligned with the markdown:

1. Confirm with the user that the frame is done.
2. Update the selected registry row:

```text
Frame Status: Finished
```

Do not mark `Finished` if:

- the Figma operation failed
- the markdown and Figma frame disagree
- the user says more changes are needed

## Write Rules

When editing local markdown:

1. Preserve existing user-written content unless the user approved replacing it.
2. Modify only the selected frame file and its selected registry row.
3. Keep table column order exactly as defined by the bundled guidance/template.
4. Use precise file edits.

## End Condition

The task is complete when:

- the selected frame was inspected in the registry
- the selected frame markdown was read and checked
- the Figma file was inspected
- the matching Figma frame exists and matches the markdown
- any interactive Figma changes were synced back into markdown
- the selected registry row is marked `Finished`

After finishing, summarize:

- Figma file link
- selected frame
- Figma frame created or updated
- markdown file path
- registry status change
- any remaining open questions
