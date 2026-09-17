# Wireframe Spec Guidance

The wireframe spec is the source of truth for the whole wireframe set. It
defines the product context, wireframe goal, frame list, frame status, and how
frames connect to each other.

Do not include backend endpoints, database tables, frontend component code, or
AI implementation details here.

---

## 1. Product Brief Guidance

Keep this section short. It should provide only the product context needed to
understand the wireframe work.

| Field | How To Fill |
|---|---|
| Product Name | Use the official or working product name. |
| Product Definition | Use one or two sentences describing what the product is. |

---

## 2. Wireframe Brief Guidance

This section explains what the wireframe set must clarify before detailed frame
specs or drawing-tool artifacts are created.

The goal is not to describe every feature. The goal is to explain what visual
decisions this wireframe set must make clear.

| Field | How To Fill |
|---|---|
| Platform | Use the target surface, such as web, responsive web, mobile app, desktop app, tablet, or other. |
| Wireframe Goal | State what decisions should be easier after this wireframe set exists. |
| Fidelity | Use low fidelity, mid fidelity, or high fidelity. For early planning, prefer low fidelity. |
| Drawing Tool | Use the tool that will hold the wireframe artifact. Use `TBD` if not chosen yet. |

---

## 3. Frame Registry Guidance

### Table Purpose

The frame registry table lists every wireframe frame that needs to be planned,
specified, drawn, reviewed, or tracked.

A frame is one specific visual thing the team may create or review. It can be a
base page, an overlay, a state of a page, or a focused detail view.

Create one row for each frame.

### Columns

Use these columns in this order:

| Column Order | Column Name |
|---|---|
| 1 | Frame ID |
| 2 | Frame Name |
| 3 | Frame Category |
| 4 | Base Frame |
| 5 | Frame Status |
| 6 | Frame Spec File |

### Column Guidance

#### Frame ID

Use a stable ID in `WF-XXX` format.

Rules:

- Start with `WF-001`.
- Increase the number by one for each new frame.
- Do not reuse old IDs after deleting or deprecating a frame.

Example:

```text
WF-001
```

#### Frame Name

Use a short human-readable name that describes what the frame shows.

Good names are specific enough to understand without opening the detailed frame
spec.

Example:

```text
Landing / Public Chat Entry
```

#### Frame Category

Use one of these values:

| Category | Use When | Example |
|---|---|---|
| Base | The frame is a primary screen or main view that can stand on its own. | Landing page, authenticated chat app |
| Overlay | The frame appears over another frame. | Signup modal, login modal, popover, action menu |
| State | The frame shows a different condition of a base frame or overlay. | Sidebar open, empty chat, loading messages, form error |
| Detail | The frame focuses on one important region inside a larger frame. | Conversation sidebar, message composer, account settings panel |

#### Base Frame

Use this column to show what frame the current frame depends on.

| Frame Category | Base Frame Value |
|---|---|
| Base | `none` |
| Overlay | The `WF-XXX` frame it appears over |
| State | The `WF-XXX` frame it is a state of |
| Detail | The `WF-XXX` frame that contains the focused region |

If the related base frame is not created yet, write short context temporarily
and replace it with a `WF-XXX` ID later.

Example:

```text
WF-001
```

#### Frame Status

Use one of these lifecycle values:

| Status | Meaning |
|---|---|
| Not Started | The frame is registered, but the frame spec file does not exist yet. |
| In Progress | The frame spec file exists, but the spec or drawing is still being worked on. |
| Finished | The frame spec is complete and the actual drawing exists in the selected drawing tool. |
| Deprecated | The frame should no longer be used, but the ID remains reserved. |

#### Frame Spec File

Use this column to link the detailed frame spec file.

Use this filename format:

```text
docs//wireframes/frames/WF-XXX-frame-name.md
```

Rules:

- Start the filename with the exact `Frame ID`.
- Convert the frame name to lowercase kebab case.
- Remove punctuation that is not needed for meaning.
- Keep the filename stable after it is created unless the frame meaning changes.
- Use `TBD` only when the file path is not decided yet.

Example:

```text
docs//wireframes/frames/WF-001-landing-public-chat-entry.md
```

### Row Examples

| Frame ID | Frame Name | Frame Category | Base Frame | Frame Status | Frame Spec File |
|---|---|---|---|---|---|
| WF-001 | Landing / Public Chat Entry | Base | none | Not Started | docs//wireframes/frames/WF-001-landing-public-chat-entry.md |
| WF-002 | Signup Modal | Overlay | WF-001 | Not Started | docs//wireframes/frames/WF-002-signup-modal.md |
| WF-003 | Chat App - Sidebar Open | State | WF-004 | Not Started | docs//wireframes/frames/WF-003-chat-app-sidebar-open.md |
| WF-004 | Authenticated Chat App | Base | none | Not Started | docs//wireframes/frames/WF-004-authenticated-chat-app.md |

---

## 4. Frame Relation Map Guidance

### Table Purpose

The frame relation map explains how users or system events move from one frame
to another.

Use this section after the frame registry has at least the important base,
overlay, and state frames listed.

Add one row per meaningful transition. Do not list tiny component-level
interactions unless they open, close, replace, or navigate to another frame.

### Columns

Use these columns in this order:

| Column Order | Column Name |
|---|---|
| 1 | From Frame |
| 2 | Trigger / Event |
| 3 | To Frame |
| 4 | What Changes |

### Column Guidance

#### From Frame

Use the frame where the user action or system event starts.

Rules:

- Use a `WF-XXX` ID from the frame registry.
- Do not use a frame name if the frame already has an ID.
- Use short context only when the frame is not registered yet.

Example:

```text
WF-001
```

#### Trigger / Event

Describe the user action or system event that causes the visual change.

Keep it product-facing. Do not describe frontend handlers or backend events.

Examples:

```text
User selects Start for Free
```

```text
Login succeeds
```

#### To Frame

Use the frame shown after the trigger/event.

Rules:

- Use a `WF-XXX` ID from the frame registry.
- The destination frame should usually already exist in the frame registry.
- If the destination frame does not exist yet, add it to the frame registry.

Example:

```text
WF-002
```

#### What Changes

Describe what visually changes after the trigger/event.

Write this as a present-tense product statement, not an implementation detail.

Examples:

- Signup modal opens over the landing frame.
- Signup modal closes and the landing frame is visible again.
- Authenticated chat app replaces the public landing frame.
- Chat sidebar changes to the open state.
- Chat settings detail frame becomes visible.

### Row Examples

| From Frame | Trigger / Event | To Frame | What Changes |
|---|---|---|---|
| WF-001 | User selects Start for Free | WF-002 | Signup modal opens over the landing frame. |
| WF-002 | User closes signup modal | WF-001 | Signup modal closes and the landing frame is visible again. |
| WF-001 | User logs in successfully | WF-004 | Authenticated chat app replaces the public landing frame. |
| WF-004 | User selects sidebar toggle | WF-003 | Chat sidebar changes to the open state. |
