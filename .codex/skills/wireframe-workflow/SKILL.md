---
name: wireframe-workflow
description: Use when a user asks what wireframe workflow skills exist, what to do next, how the wireframe documentation/drawing process works, or wants a menu/overview to choose the right wireframe skill. Acts as the root guide for wireframe-spec-init, wireframe-frame-register, wireframe-frame-write, wireframe-relation-register, and wireframe-frame-draw-figma.
---

# Wireframe Workflow

Use this root skill to orient the user and route them to the right wireframe
skill.

This skill explains the workflow. It should not duplicate the detailed behavior
of the child skills.

## Skill Map

Use these skills in this general order:

| Order | Skill | Purpose |
|---|---|---|
| 1 | `wireframe-spec-init` | Create `docs/central/wireframes/wireframe-spec.md` and fill sections 1-2 from product brief plus user answers. |
| 2 | `wireframe-frame-register` | Add a new row to the `Frame Registry` table. |
| 3 | `wireframe-frame-write` | Create or continue one frame markdown file from a registered frame row. |
| 4 | `wireframe-relation-register` | Add rows to the `Frame Relation Map` manually or from suggestions. |
| 5 | `wireframe-frame-draw-figma` | Draw or sync a Figma wireframe from a completed frame markdown spec. |

## User-Facing Menu

When the user asks what they can do, show this concise menu:

```text
Wireframe workflow options:

1. Initialize wireframe spec
   Use: wireframe-spec-init
   Creates docs/central/wireframes/wireframe-spec.md and fills sections 1-2.

2. Register a frame
   Use: wireframe-frame-register
   Adds a row to the Frame Registry.

3. Write a frame markdown spec
   Use: wireframe-frame-write
   Creates or continues a frame file with visual purpose, regions, and components.

4. Register frame relations
   Use: wireframe-relation-register
   Adds rows to the Frame Relation Map manually or from suggestions.

5. Draw in Figma
   Use: wireframe-frame-draw-figma
   Draws or syncs a Figma frame from an In Progress frame markdown file.
```

Then ask what they want to do next.

## Routing Rules

Route by intent:

| User Intent | Use Skill |
|---|---|
| Start wireframe docs from product brief | `wireframe-spec-init` |
| Add a new frame row | `wireframe-frame-register` |
| Create/fill/edit a frame markdown file | `wireframe-frame-write` |
| Add or suggest frame relation rows | `wireframe-relation-register` |
| Draw/update/sync a Figma wireframe | `wireframe-frame-draw-figma` |
| Ask what skills exist or what comes next | `wireframe-workflow` |

If the user asks for a child skill by name, use that skill directly.

If the user describes an action without naming a skill, recommend the matching
skill and ask for confirmation before doing anything destructive or file-writing.

## File-State Orientation

When the user is unsure what to do next, inspect the current file state:

```text
docs/central/product-brief.md
docs/central/wireframes/wireframe-spec.md
docs/central/wireframes/frames/
```

Use this decision guide:

| Current State | Recommended Next Skill |
|---|---|
| Product brief missing | Tell the user to create product brief first. |
| Product brief exists, wireframe spec missing | `wireframe-spec-init` |
| Wireframe spec exists, no real frame rows | `wireframe-frame-register` |
| Frame rows exist with `Not Started` | `wireframe-frame-write` |
| Frame files exist and relations are missing | `wireframe-relation-register` |
| Frame files are In Progress and ready to draw | `wireframe-frame-draw-figma` |

## Activation Help

Use simple activation prompts when guiding the user:

```text
Use wireframe-spec-init to start the wireframe spec.
```

```text
Use wireframe-frame-register to add a frame row.
```

```text
Use wireframe-frame-write to write the selected frame markdown file.
```

```text
Use wireframe-relation-register to add frame relation rows.
```

```text
Use wireframe-frame-draw-figma to draw the frame in Figma.
```

## Communication Rules

- Keep explanations short and option-oriented.
- Do not overwhelm the user with every detail from every child skill.
- Mention the next likely skill, the file it affects, and why.
- If the user is confused, show the menu and recommend one next step.
- If the user asks for a deep explanation, explain the workflow from spec to
  frame rows to frame markdown to relation rows to Figma drawing.

## Workflow Summary

Use this explanation when the user asks for the whole picture:

```text
The wireframe workflow has five steps.

First, initialize the central wireframe spec from the product brief.
Second, register each frame that needs to exist.
Third, write the markdown spec for each frame.
Fourth, register how frames connect in the relation map.
Fifth, draw or sync the ready frame specs in Figma.
```
