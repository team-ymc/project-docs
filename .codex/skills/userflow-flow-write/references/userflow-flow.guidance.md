# Userflow Flow Guidance

Use this guidance when creating or updating an individual userflow file.

An individual userflow file describes one straight product flow from a clear
beginning state to a clear ending state. Alternate flows, exception flows, and
subflows should be separate userflow files when they need to be tracked.

Do not include backend endpoints, database tables, frontend component code, API
details, or hidden implementation behavior here.

---

## Context To Check Before Writing

Userflow files should usually live at:

```text
docs//userflows/flows/UF-XXX-flow-name.md
```

Each userflow file should correspond to one row in:

```text
docs//userflows/userflow-spec.md
```

Use nearby product documents intentionally. The userflow spec tells you what
flow this file represents. The wireframe spec tells you which visual states can
support the flow.

### Always Check

Before filling a userflow file, read the userflow spec when it exists:

- `docs//userflows/userflow-spec.md`

Use it to find:

- the matching Flow Registry row
- the flow type
- the base flow
- the flow status and file path

If the userflow spec does not exist yet, write only from the information the
user has provided and leave central-spec alignment for later.

### Check When Relevant

Read the existing userflow file when updating a flow.

Read the base flow file when:

- `Base Flow` is not `none`
- the flow is an alternate flow, exception flow, or subflow
- the new flow must stay consistent with a larger or related flow

Read the wireframe spec when filling `Wireframe References`:

- `docs//wireframes/wireframe-spec.md`

Use it to identify useful `WF-XXX` frames and relation-map transitions.

Read specific wireframe frame files only when:

- the registry row and frame name are not enough to explain relevance
- a flow step depends on visible regions/components
- the wireframe sequence needs to be checked against actual frame content

Do not read every wireframe frame file by default. Start with the wireframe spec
registry and relation map, then open only the frames needed to understand this
flow.

---

## Relationship To The Userflow Spec

The central spec owns:

- Flow ID
- Flow Name
- User Type
- Flow Type
- Base Flow
- Flow Status
- Flow Spec File

Do not duplicate those fields as separate metadata sections in the flow file.
The flow file should focus on the actual straight flow.

The title should match the registry row:

```text
# Userflow: UF-XXX <Flow Name>
```

---

## 1. Flow Boundary Guidance

The flow boundary defines the scope of the straight flow.

Use it to prevent the file from drifting into adjacent alternate paths,
exception paths, or subflows.

### Begins

Describe the concrete user/product state where this flow starts.

Good beginnings are observable and specific.

Examples:

- The guest is viewing the public chat entry screen.
- The auth modal is open and ready for email input.
- The authenticated user is viewing the empty chat app.

### Ends

Describe the observable user/product state that means this flow is complete.

Examples:

- The account setup page is shown.
- The user is signed in and reaches the authenticated chat app.
- The completed assistant response is visible.

### Assumes

List conditions that must stay true for this straight flow to hold.

Use assumptions instead of adding branches.

Examples:

- The email verification code is valid.
- The message sends successfully.
- No network or rate-limit error occurs.

---

## 2. Flow Story Guidance

Write one short paragraph that explains the flow in human terms.

Include:

- the user's situation
- what the user is trying to complete
- the straight path the user follows
- the outcome reached inside this flow boundary

Do not turn the story into a step list. Do not include implementation details.

Example:

```text
An authenticated user starts from the empty chat app, writes a message, sends it, watches the assistant response begin, and ends with a completed exchange ready for follow-up.
```

---

## 3. Flow Steps Guidance

Flow steps describe the meaningful progression of the straight flow.

Each step should represent a user-understandable state or state change, not
every click, keystroke, validation check, or implementation event.

Use this format:

```text
1. **<Step name>**  
   <One sentence describing the state or meaningful change at this point in the flow.>
```

Rules:

- The first step should align with `Begins`.
- The last step should align with `Ends`.
- Keep the sequence linear.
- Combine minor UI interactions into a larger meaningful step.
- Move meaningful alternate, exception, or subflow situations to `Separate Flow Candidates`.

Example step names:

- Ready To Chat
- Message Drafted
- Message Submitted
- Response Streaming
- Exchange Complete

---

## 4. Wireframe References Guidance

Wireframe references connect the userflow to visual planning work.

Use wireframes as supporting context. Do not duplicate layout regions,
component inventories, or markdown sketches from wireframe files.

### Wireframe Flow

Use `Wireframe Flow` to show the visual sequence that supports the userflow.

This sequence does not need to map one-to-one to flow steps. Include only the
wireframes that help review or understand the flow.

Format:

```text
WF-XXX <Frame Name>
  -> WF-XXX <Frame Name>
  -> WF-XXX <Frame Name>
```

If no useful wireframe exists yet, write:

```text
TBD
```

### Referenced Wireframes

Use the reference table to explain why each wireframe matters.

| Wireframe | Relevance |
|---|---|
| WF-XXX <Frame Name> | Explain how this frame helps review or understand the flow. |

Rules:

- Use exact `WF-XXX` IDs from `docs//wireframes/wireframe-spec.md`.
- Include the frame name for readability.
- Keep relevance statements short.
- Explain the frame's role in the flow, not its full visual structure.
- Read a referenced frame file only when the spec row is not enough.

Example:

| Wireframe | Relevance |
|---|---|
| WF-008 Authenticated Chat App | Shows the starting chat shell for the flow. |
| WF-012 Chat App - Conversation Complete | Shows the completed conversation state. |

---

## 5. Separate Flow Candidates Guidance

Use this section for situations that should not be handled inside the current
straight flow, but may deserve their own registered userflow.

Good candidates include:

- alternate paths to a similar outcome
- exception or recovery paths
- subflows that support the current flow
- product decisions that would change the current flow if chosen

Keep each candidate short. Do not describe the full candidate flow here.

Examples:

- Message send failure
- Assistant response timeout
- User pauses assistant response
- User starts a new chat instead of sending in the current conversation

If there are no obvious candidates, write:

```text
- None for now.
```
