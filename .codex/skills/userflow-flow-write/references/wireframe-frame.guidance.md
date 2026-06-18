# Wireframe Frame Guidance

Use this guidance when creating or updating an individual wireframe frame file.

An individual frame file describes one visual frame: its purpose, large layout
regions, visible components, and rough markdown sketch.

Do not include backend endpoints, database tables, frontend component code, or
AI implementation details here.

---

## Context To Check Before Writing

Frame spec files should usually live at:

```text
docs/central/wireframes/frames/WF-XXX-frame-name.md
```

Each frame spec should correspond to one row in:

```text
docs/central/wireframes/wireframe-spec.md
```

Use the surrounding wireframe documents intentionally. They tell you what this
frame is supposed to be, how it relates to other frames, and which nearby frames
must stay visually consistent.

### Always Check

Before filling a frame file, read:

- `docs/central/wireframes/wireframe-spec.md`

Use it to find:

- the matching Frame Registry row
- the frame category
- the base frame
- the frame status and file path
- related transitions in the Frame Relation Map

### Check When Relevant

Read the selected frame file when updating an existing frame.

Read the base frame file when:

- `Base Frame` is not `none`
- the frame is an overlay, state, or detail
- regions/components should stay consistent with a related frame

Read directly related frames when:

- the Frame Relation Map shows a transition into or out of this frame
- the frame is part of a state sequence
- a component or region should persist across several frames

Do not read every frame file by default. Start with the registry and relation
map, then open only the files needed to keep this frame accurate.

---

## Frame Title Guidance

The title should match the frame row in the wireframe spec registry.

Use this format:

```text
# Frame: WF-XXX <Frame Name>
```

Rules:

- Use the exact `Frame ID` from the registry.
- Use the exact or near-exact `Frame Name` from the registry.
- Keep the title stable unless the frame meaning changes.

Example:

```text
# Frame: WF-001 Public Chat Entry
```

---

## 1. Visual Purpose Guidance

Describe what the frame must make clear to the user at a glance.

Keep this short. The visual purpose should explain why this frame exists as a
separate visual frame.

Example:

```text
This frame should make it clear that unauthenticated users land in a chat-style entry experience before authentication.
```

---

## 2. Layout Regions Guidance

### Table Purpose

The layout regions table defines the large visual areas inside the frame.

Regions are structural areas such as top bar, sidebar, main content, modal
panel, footer, chat area, or composer area. They are not every UI element.

### Columns

Use these columns in this order:

| Column Order | Column Name |
|---|---|
| 1 | Region ID |
| 2 | Region Name |
| 3 | Parent Region |
| 4 | Placement Notes |
| 5 | Region Purpose |

### Column Guidance

#### Region ID

Use a stable ID in `R#` format.

Rules:

- Every frame starts with `R0`.
- `R0` is always `Frame Root`.
- Use `R1`, `R2`, `R3`, and so on for additional regions.

#### Region Name

Use a short human-readable name for the visual region.

Example:

```text
Top Bar
```

#### Parent Region

Use the region ID that visually contains this region.

Rules:

- `R0` uses `None`.
- Every other region should use an existing region ID.

#### Placement Notes

Describe where the region appears relative to its parent or nearby regions.

Examples:

- Positioned at the top of the frame.
- Fixed vertically on the left edge.
- Centered inside the modal body.

#### Region Purpose

Explain why the region exists in the frame.

Example:

```text
Holds product identity and auth actions.
```

### Row Examples

| Region ID | Region Name | Parent Region | Placement Notes | Region Purpose |
|---|---|---|---|---|
| R0 | Frame Root | None | Root container for the whole frame | Root container for this frame |
| R1 | Top Bar | R0 | Positioned at the top of the frame | Holds product identity and actions |
| R2 | Chat Entry Area | R0 | Centered below the top bar | Presents the chat starting point |

---

## 3. Component Inventory Guidance

### Table Purpose

The component inventory lists meaningful visible UI components inside the frame.

Components are product/UI pieces, not frontend implementation components.

### Columns

Use these columns in this order:

| Column Order | Column Name |
|---|---|
| 1 | Component ID |
| 2 | Component Name |
| 3 | Region ID |
| 4 | Component Type |
| 5 | Component Purpose |

### Column Guidance

#### Component ID

Use a stable ID in `C#` format.

Example:

```text
C1
```

#### Component Name

Use a short human-readable name for the component.

Example:

```text
Login Button
```

#### Region ID

Use the ID of the region that contains this component.

Example:

```text
R1
```

#### Component Type

Use a recognizable UI type. This is flexible, not a strict enum.

Example values:

- Text
- Button
- Icon Button
- Text Input
- Textarea
- Message Bubble
- Spinner
- Separator
- Modal
- List
- List Item

#### Component Purpose

Explain why the component exists in the frame.

Example:

```text
Lets returning users open the auth flow.
```

---

## 4. Markdown Sketch Guidance

Create a rough text sketch of the frame.

Use the accepted regions and components as the source of truth. The sketch
should show approximate placement and hierarchy, not exact styling.

Rules:

- Keep the sketch low fidelity.
- Show only meaningful visible regions and components.
- Do not duplicate the tables.
- Do not introduce components that are missing from the inventory.

Use a fenced `text` block.

Example:

```text
┌──────────────────────────────┐
│ Top Bar                      │
│                              │
│        Chat Entry Area       │
│        [Message composer]    │
└──────────────────────────────┘
```
