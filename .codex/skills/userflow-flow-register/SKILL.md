---
name: userflow-flow-register
description: Use when a user wants to add one or more rows to the Flow Registry table in docs/central/userflows/userflow-spec.md, either manually or from AI suggestions based on docs/central/wireframes/wireframe-spec.md and relevant WF-XXX frame files. Starts by asking whether to register manually or generate suggestions, validates flow names, user types, flow types, base flows, IDs, statuses, and file paths, then writes only approved rows.
---

# Userflow Flow Register

Use this skill to add rows to the `## 3. Flow Registry` table in the central
userflow spec.

Default target file:

```text
docs/central/userflows/userflow-spec.md
```

Bundled userflow guidance:

```text
references/userflow-spec.guidance.md
```

Bundled wireframe guidance:

```text
references/wireframe-spec.guidance.md
```

Bundled paths are relative to this skill directory. Do not assume the skill is
installed at a specific project path.

The skill is complete when approved flow registry rows have been added, or when
the user chooses not to add any rows.

## Hard Stop Rule

Before doing anything else, check whether this file exists:

```text
docs/central/userflows/userflow-spec.md
```

If it does not exist, stop immediately and tell the user:

```text
I cannot register userflows yet because docs/central/userflows/userflow-spec.md does not exist.
Initialize the userflow spec first, then I can add flow rows.
```

Do not create `userflow-spec.md` in this skill.

## Required Reference Rule

Always read these files before adding flow rows:

```text
docs/central/userflows/userflow-spec.md
references/userflow-spec.guidance.md
```

Use `references/userflow-spec.guidance.md` as the authority for how to fill Flow
Registry columns.

For suggestion mode, also read:

```text
docs/central/wireframes/wireframe-spec.md
references/wireframe-spec.guidance.md
```

Read individual `docs/central/wireframes/frames/WF-XXX-frame-name.md` files only
when their visual purpose, components, or sketch are needed to make a good
suggestion.

## Scope Rule

This skill only adds rows to the Flow Registry table.

Do not:

- create userflow files
- fill individual userflow-flow markdown files
- edit Product Brief or Userflow Brief sections
- edit wireframe files
- change existing flow rows unless the user explicitly asks for a correction

## Start Flow

Read the Flow Registry table and show it to the user in a compact table:

```text
Flow ID | Flow Name | User Type | Flow Type | Base Flow | Flow Status | Flow Spec File
```

Then ask whether the user wants:

```text
manual add
```

or:

```text
suggestions from wireframes
```

Do not continue until the user chooses a mode.

## New Row Values

Every new row must use these columns:

```text
Flow ID | Flow Name | User Type | Flow Type | Base Flow | Flow Status | Flow Spec File
```

The user or selected suggestion provides:

```text
Flow Name
User Type
Flow Type
Base Flow
```

The skill fills automatically:

```text
Flow ID: next available UF-XXX ID
Flow Status: Not Started
Flow Spec File: TBD
```

## Flow ID Rule

Find all existing `UF-XXX` IDs in the Flow Registry table.

Choose the next highest number.

Examples:

```text
Existing: UF-001, UF-002
New: UF-003
```

```text
Existing: UF-001, UF-003
New: UF-004
```

Do not reuse deprecated, deleted, or missing IDs.

When writing multiple selected suggestions, assign IDs sequentially after the
last existing `UF-XXX`.

## Existing Placeholder Row Rule

If the Flow Registry table only contains the template placeholder row, replace
that placeholder row with the first real row.

A placeholder row usually contains values like:

```text
<Flow name>
<User type>
<Main Flow / Alternate Flow / Exception Flow / Subflow>
<none or UF-XXX>
```

If the table already contains real flow rows, append new rows after the last
existing flow row.

## Validation Rules

Validate every proposed row before asking for write approval.

Required checks:

- `Flow Name` describes a whole straight flow, not a single screen or tiny step.
- `User Type` is product-facing, such as `Guest` or `Authenticated User`.
- `User Type` is not an implementation actor such as backend service, database,
  frontend app, API provider, or hidden system process.
- `Flow Type` is one of `Main Flow`, `Alternate Flow`, `Exception Flow`, or
  `Subflow`.
- `Base Flow` follows the flow type rule.
- The row is not a duplicate of an existing Flow Registry row.
- `Flow Status` is `Not Started`.
- `Flow Spec File` is `TBD`.

Base flow rules:

| Flow Type | Base Flow Value |
|---|---|
| Main Flow | `none` |
| Alternate Flow | The `UF-XXX` flow it is an alternate route for. |
| Exception Flow | The `UF-XXX` flow where the exception or blocked state occurs. |
| Subflow | The `UF-XXX` larger flow that contains or uses this smaller flow. |

If an `Alternate Flow`, `Exception Flow`, or `Subflow` has no valid base flow
yet, explain the issue and ask whether to register a suitable main/base flow
first.

## Manual Add Mode

Help the user add one flow row interactively.

Explain the process before asking questions:

```text
I will add one new row to the Flow Registry.
I will determine the next Flow ID automatically.
I will ask for Flow Name, User Type, Flow Type, and Base Flow.
Flow Status will be Not Started, and Flow Spec File will be TBD.
```

Ask for one value at a time.

### Step 1: Flow Name

Ask what the flow should be called.

The name should be concise and describe the whole straight flow.

Good examples:

```text
Email Signup
Google Signup
Email Login
Authenticated Conversation
```

If the user gives a screen name, one-step action, or vague name, suggest a
clearer whole-flow name and ask for confirmation.

### Step 2: User Type

Ask which user type owns this flow.

This is an open value. Recommend likely values from the product context and
existing registry rows.

For this product, likely values are:

```text
Guest
Authenticated User
```

### Step 3: Flow Type

Ask which flow type applies.

Allowed values:

```text
Main Flow
Alternate Flow
Exception Flow
Subflow
```

Briefly explain each option from `references/userflow-spec.guidance.md` if the
user is unsure.

### Step 4: Base Flow

Ask for the base flow based on the chosen flow type.

Rules:

```text
Main Flow -> none
Alternate Flow -> UF-XXX flow it is an alternate route for
Exception Flow -> UF-XXX flow where the exception or blocked state occurs
Subflow -> UF-XXX larger flow that contains or uses this smaller flow
```

If the user does not know the exact flow ID, inspect the existing Flow Registry
rows and recommend likely options.

## Suggestion Mode

Use suggestion mode to propose useful userflow rows from the existing wireframe
work.

Suggestion mode must inspect:

1. Existing Flow Registry rows.
2. `docs/central/wireframes/wireframe-spec.md`.
3. The wireframe Frame Registry.
4. The wireframe Frame Relation Map.
5. Relevant wireframe frame files when registry and relation rows are not enough.

Do not read every frame file by default. Read only the frames that support a
likely candidate flow.

### Suggestion Heuristics

Suggest userflows by grouping related wireframe frames into product journeys.

Do not suggest one userflow per wireframe state by default.

Good sources for suggestions:

- A sequence of public/auth frames that form a signup, login, verification, or
  account setup journey.
- A sequence of authenticated chat states that form a conversation journey.
- A meaningful alternate route visible in components, such as Google auth.
- A visible error, blocked, or recovery state.
- A focused reusable interaction that supports a larger flow.

Flow type guidance:

- Use `Main Flow` for the default successful path of a major user journey.
- Use `Alternate Flow` for a distinct valid route to a similar outcome.
- Use `Exception Flow` only when wireframes or userflow context support a
  failure, blocked, or recovery path.
- Use `Subflow` for a smaller path that belongs inside a larger flow.

Name suggestions should be concise whole-flow titles, such as:

```text
Email Signup
Google Signup
Email Login
Authenticated Conversation
```

Avoid names like:

```text
User clicks Continue
Auth Modal
Message Entered
Assistant Loading
```

### Suggestion Format

Show suggestions as a table:

```text
Suggestion ID | Flow Name | User Type | Flow Type | Base Flow | Wireframe Evidence | Reason
```

Use `Wireframe Evidence` to list supporting frames or frame ranges, such as:

```text
WF-002 -> WF-006
WF-008 -> WF-012
```

Keep reasons short and concrete.

Ask which suggestions the user wants to add.

The user may choose:

- all
- none
- specific suggestion IDs
- revised versions of suggestions

Before writing, assign final `UF-XXX` IDs to selected suggestions, validate all
rows again, and show the exact rows that will be added.

## Draft And Approval Rule

Before editing the file, show the exact row or rows to be added:

```text
| UF-XXX | <Flow Name> | <User Type> | <Flow Type> | <Base Flow> | Not Started | TBD |
```

Ask for approval before writing.

Only edit `docs/central/userflows/userflow-spec.md` after approval.

## Write Rule

When writing:

1. Preserve all existing content.
2. Modify only the Flow Registry table row area.
3. Replace the placeholder row only when it is the only registry row.
4. Otherwise append approved rows after the last existing flow row.
5. Do not add duplicate rows.

## End Conditions

The skill ends when one of these is true:

- An approved manual row has been written.
- Approved suggestion rows have been written.
- The user chooses not to add any rows.
- No valid suggestions exist and the user does not want to add manually.
- Required files are missing, so the skill cannot proceed.

After writing, summarize:

- target file path
- rows added
- rows skipped or rejected, if any
- any likely follow-up flows that remain unregistered
