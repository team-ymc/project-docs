---
name: userflow-flow-write
description: Use when a user wants to create, write, continue, or revise an individual docs//userflows/flows/UF-XXX-flow-name.md userflow file from a row in docs//userflows/userflow-spec.md. Selects a registered flow, creates the flow file from assets/userflow-flow.template.md when needed, updates the registry status/path, inspects related wireframes when available, identifies missing wireframe needs, and interactively fills Flow Boundary, Flow Story, Flow Steps, Wireframe References, and Separate Flow Candidates using bundled guidance with frequent user review.
---

# Userflow Flow Write

Use this skill to create or continue one individual userflow file.

Default registry file:

```text
docs//userflows/userflow-spec.md
```

Default flow directory:

```text
docs//userflows/flows
```

Default wireframe spec:

```text
docs//wireframes/wireframe-spec.md
```

Bundled flow template:

```text
assets/userflow-flow.template.md
```

Bundled flow guidance:

```text
references/userflow-flow.guidance.md
```

Bundled userflow spec guidance:

```text
references/userflow-spec.guidance.md
```

Bundled wireframe guidance:

```text
references/wireframe-spec.guidance.md
references/wireframe-frame.guidance.md
```

Bundled paths are relative to this skill directory. Do not assume the skill is
installed at a specific project path.

The skill is complete when the selected flow file exists and sections 1-5 are
filled enough for the user to review the straight flow, connect it to available
wireframes, and see which separate flows or missing wireframes may need follow-up.

## Hard Stop Rule

Before doing anything else, check whether this file exists:

```text
docs//userflows/userflow-spec.md
```

If it does not exist, stop immediately and tell the user:

```text
I cannot author a userflow file yet because docs//userflows/userflow-spec.md does not exist.
Initialize the userflow spec and register a flow first, then I can write the flow file.
```

Do not create `userflow-spec.md` in this skill.

## Required Reference Rule

Always read these files before creating or editing a userflow file:

```text
docs//userflows/userflow-spec.md
assets/userflow-flow.template.md
references/userflow-flow.guidance.md
references/userflow-spec.guidance.md
```

Use `references/userflow-flow.guidance.md` as the authority for filling:

- title
- Flow Boundary
- Flow Story
- Flow Steps
- Wireframe References
- Separate Flow Candidates

Use `references/userflow-spec.guidance.md` only for registry status/path rules
and Flow Registry column meanings.

When wireframes exist, also read:

```text
docs//wireframes/wireframe-spec.md
references/wireframe-spec.guidance.md
references/wireframe-frame.guidance.md
```

If the wireframe spec does not exist, continue without it and make the
wireframe sections `TBD` where appropriate.

## Flow Selection Flow

Read the `## 3. Flow Registry` table from:

```text
docs//userflows/userflow-spec.md
```

Show the registry rows to the user in a compact table with:

```text
Flow ID | Flow Name | User Type | Flow Type | Base Flow | Flow Status | Flow Spec File
```

Ask which flow the user wants to handle.

If there is only one real flow row, recommend it but still ask for confirmation.

Do not continue until the user selects one flow.

## Status Behavior

### Not Started

If the selected flow status is `Not Started`:

1. Determine the flow file path.
2. Create the flow file from `assets/userflow-flow.template.md`.
3. Replace the title with `# Userflow: UF-XXX <Flow Name>`.
4. Update the selected registry row:
   - `Flow Status` becomes `In Progress`
   - `Flow Spec File` becomes the created file path
5. Inspect userflow and wireframe context before drafting sections.

### In Progress

If the selected flow status is `In Progress`:

1. Read the flow file from the `Flow Spec File` path.
2. Show the current markdown version or a concise section summary to the user.
3. Inspect current userflow and wireframe context.
4. Ask what the user wants to work on:
   - Flow Boundary
   - Flow Story
   - Flow Steps
   - Wireframe References
   - Separate Flow Candidates
   - general cleanup

If `Flow Spec File` is `TBD` while status is `In Progress`, generate the
expected path, create the file if needed, and update the registry path after
confirming with the user.

### Finished

If the selected flow status is `Finished`, do not edit immediately.

Show the current flow file and ask whether the user wants to revise it. If the
user approves revision, change the registry status to `In Progress` before
editing.

### Deprecated

If the selected flow status is `Deprecated`, stop unless the user explicitly
asks to revive or inspect it.

## Flow File Path Rule

If `Flow Spec File` already contains a real path, use that path.

If `Flow Spec File` is `TBD`, generate the path from the selected flow row:

```text
docs//userflows/flows/UF-XXX-flow-name.md
```

Rules:

- Start the filename with the exact `Flow ID`.
- Convert the flow name to lowercase kebab case.
- Remove punctuation that is not needed for meaning.
- Keep the path stable after creation.

## Base Flow Context Rule

If the selected row has a `Base Flow` value other than `none`:

1. Find that base flow row in the Flow Registry.
2. If its `Flow Spec File` path exists, read it.
3. Briefly summarize the base flow's boundary, steps, and wireframe references.
4. Ask how the selected flow differs from or depends on the base flow.

Use this to help alternate flows, exception flows, and subflows stay consistent
with their related main flow.

If the base flow file does not exist yet, continue without it and mention that
base-flow comparison is unavailable.

## Wireframe Context Rule

Wireframes are important context for this skill, but they are not always
complete before a userflow is written.

When `docs//wireframes/wireframe-spec.md` exists:

1. Read the Frame Registry and Frame Relation Map.
2. Identify likely relevant `WF-XXX` frames for the selected flow.
3. Read relevant wireframe frame files only when the registry and relation map
   are not enough to understand the visual state.
4. Summarize useful wireframe evidence before drafting flow sections.

When likely wireframes are missing:

1. Tell the user what kind of wireframe seems needed.
2. Explain which flow section or step needs it.
3. Use `TBD` in the Wireframe References section when the user wants to proceed.
4. Suggest registering or writing the missing wireframe later; do not create it
   in this skill.

When no wireframe spec exists, continue with user-provided context and write
wireframe references as `TBD`.

## Context Review Rule

Before filling sections, show a concise context summary:

```text
Selected flow:
Base flow context:
Relevant existing wireframes:
Possible missing wireframes:
Recommended authoring order:
```

Ask the user to confirm or correct the context summary before drafting section
content.

Do not force the user to fill sections 1-5 in a blind linear order when
wireframe context is needed first. It is valid to discuss wireframes before
drafting Flow Boundary or Flow Steps.

## Interactive Filling Flow

Work section by section, but stay flexible when wireframe context changes the
draft.

For each section:

1. Ask focused questions.
2. Offer a proposed draft based on the selected flow row, base flow context,
   user answers, and available wireframes.
3. Ask the user what is missing, wrong, or unnecessary.
4. Write only after the user accepts the section draft.

Do not ask the user to produce a full section alone. Help propose concise
content and make the user review it.

### 1. Flow Boundary

Follow `references/userflow-flow.guidance.md` exactly.

Help the user fill:

```text
Begins
Ends
Assumes
```

Use available wireframes to make boundary states concrete.

Ask:

- what state the flow begins in
- what visible state means the flow is complete
- what assumptions keep this flow straight

If no wireframe exists for a beginning or ending state, mention the missing
wireframe need before finalizing the boundary.

Show the proposed boundary and ask for approval before writing.

### 2. Flow Story

Write one short paragraph.

The story should describe:

- the user's situation
- what the user is trying to complete
- the straight path the user follows
- the outcome reached inside the boundary

Do not duplicate registry metadata. Do not turn the story into a step list.

Show the proposed story and ask for approval before writing.

### 3. Flow Steps

Build the step sequence interactively.

Use meaningful user-understandable states or state changes. Do not list every
click, keystroke, validation check, or implementation event.

Suggested workflow:

1. Ask for the main milestones in the straight flow.
2. Propose concise step names and one-sentence descriptions.
3. Compare the first step to `Begins` and the last step to `Ends`.
4. Check whether available wireframes support the important visual states.
5. Identify any missing wireframes that would help explain the step sequence.
6. Ask the user what is missing, wrong, or unnecessary.
7. Write only after the user accepts the steps.

If an alternate path, exception, or subflow appears while drafting steps, move it
to `Separate Flow Candidates` instead of adding it to the main sequence.

### 4. Wireframe References

Fill this section from the available wireframe context.

It has two parts:

```text
Wireframe Flow
Referenced Wireframes
```

For `Wireframe Flow`, show a useful visual sequence of existing frames. It does
not need to map one-to-one to flow steps.

For `Referenced Wireframes`, list each frame and explain how it helps review or
understand the userflow.

If a needed wireframe is missing, use `TBD` and describe what is needed in the
reference table or in `Separate Flow Candidates`, depending on the situation.

Show the proposed wireframe references and ask for approval before writing.

### 5. Separate Flow Candidates

Use this section for situations that should not be handled inside the current
straight flow, but may deserve their own registered flow.

Good candidates include:

- alternate paths to a similar outcome
- exception or recovery paths
- subflows that support the current flow
- missing userflow coverage discovered while writing
- product decisions that would change the current flow if chosen

Ask whether any candidates should be listed.

Recommend candidates from:

- the selected flow type
- base flow context
- wireframe relation-map transitions not covered by this flow
- missing wireframe needs discovered during drafting

If there are no obvious candidates, write:

```text
- None for now.
```

Show the proposed candidate list and ask for approval before writing.

## Write Rules

Before editing files, show the draft change and ask for approval.

When writing:

1. Preserve existing user-written content unless the user approved replacing it.
2. Modify only the selected flow file and its selected registry row.
3. Keep section headings exactly as defined by the bundled flow template.
4. Keep the flow file focused on one straight flow.
5. Use precise file edits.

## End Condition

The task is complete when:

- the selected flow file exists
- the registry row points to the flow file
- the registry status is `In Progress`, unless the user explicitly keeps another status
- `## 1. Flow Boundary` is filled
- `## 2. Flow Story` is filled
- `## 3. Flow Steps` has accepted steps
- `## 4. Wireframe References` is filled with existing frames or `TBD` notes
- `## 5. Separate Flow Candidates` is filled

After writing, summarize:

- selected flow
- flow file path
- registry status/path update
- filled sections
- wireframes used
- missing wireframes or follow-up flow candidates
