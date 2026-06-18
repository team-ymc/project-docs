# Userflow Spec Guidance

The userflow spec is the source of truth for the whole userflow set. It defines
the product context, userflow goal, flow list, flow status, and where detailed
flow files live.

Do not include backend endpoints, database tables, frontend component code, API
details, or hidden implementation behavior here.

Detailed flow boundaries, steps, wireframe references, and separate flow
candidates belong in individual flow files.

---

## 1. Product Brief Guidance

Keep this section short. It should provide only the product context needed to
understand the userflow work.

| Field | How To Fill |
|---|---|
| Product Name | Use the official or working product name. |
| Product Definition | Use one or two sentences describing what the product is. |

---

## 2. Userflow Brief Guidance

This section explains what the userflow set must clarify before detailed flow
files, wireframes, or implementation planning are created.

The goal is not to describe every feature. The goal is to explain which user
journeys and product decisions this userflow set must make clear.

| Field | How To Fill |
|---|---|
| Userflow Goal | State which user journeys should be clarified and what product decisions the flow set should help the team make. |

---

## 3. Flow Registry Guidance

### Table Purpose

The flow registry table lists every userflow that needs to be planned,
specified, reviewed, or tracked.

A flow is one straight path through the product. If a path branches, fails, or
becomes a reusable subtask, register that situation as a separate flow when it
needs to be tracked.

Create one row for each flow.

### Columns

Use these columns in this order:

| Column Order | Column Name |
|---|---|
| 1 | Flow ID |
| 2 | Flow Name |
| 3 | User Type |
| 4 | Flow Type |
| 5 | Base Flow |
| 6 | Flow Status |
| 7 | Flow Spec File |

### Column Guidance

#### Flow ID

Use a stable ID in `UF-XXX` format.

Rules:

- Start with `UF-001`.
- Increase the number by one for each new flow.
- Do not reuse old IDs after deleting or deprecating a flow.

Example:

```text
UF-001
```

#### Flow Name

Use a short human-readable name that describes the whole straight flow.

Good names are specific enough to understand without opening the detailed flow
file.

Example:

```text
Google Signup
```

#### User Type

Use a concise product-facing user type.

This is an open value, not a fixed enum. For this product, likely values are:

```text
Guest
Authenticated User
```

Do not use implementation actors such as backend service, database, frontend
app, or API provider.

#### Flow Type

Use one of these values:

| Flow Type | Use When |
|---|---|
| Main Flow | The default successful path for an important user goal. |
| Alternate Flow | A valid different path toward the same or similar goal. |
| Exception Flow | A path caused by an error, blocked state, failed validation, or recovery need. |
| Subflow | A smaller reusable or supporting path that belongs inside a larger flow. |

#### Base Flow

Use this column to show what flow the current flow depends on.

| Flow Type | Base Flow Value |
|---|---|
| Main Flow | `none` |
| Alternate Flow | The `UF-XXX` flow it is an alternate route for. |
| Exception Flow | The `UF-XXX` flow where the exception or blocked state occurs. |
| Subflow | The `UF-XXX` larger flow that contains or uses this smaller flow. |

If the related base flow is not registered yet, write short context temporarily
and replace it with a `UF-XXX` ID later.

Example:

```text
UF-001
```

#### Flow Status

Use one of these lifecycle values:

| Status | Meaning |
|---|---|
| Not Started | The flow is registered, but the flow file does not exist yet. |
| In Progress | The flow file exists, but it is still being worked on. |
| Finished | The flow file is complete enough to guide design or implementation. |
| Deprecated | The flow should no longer be used, but the ID remains reserved. |

#### Flow Spec File

Use this column to link the detailed flow file.

Use this filename format:

```text
docs/central/userflows/flows/UF-XXX-flow-name.md
```

Rules:

- Start the filename with the exact `Flow ID`.
- Convert the flow name to lowercase kebab case.
- Remove punctuation that is not needed for meaning.
- Keep the filename stable after it is created unless the flow meaning changes.
- Use `TBD` only when the file path is not decided yet.

Example:

```text
docs/central/userflows/flows/UF-001-guest-starts-first-chat.md
```

### Row Examples

| Flow ID | Flow Name | User Type | Flow Type | Base Flow | Flow Status | Flow Spec File |
|---|---|---|---|---|---|---|
| UF-001 | Email Signup | Guest | Main Flow | none | Not Started | docs/central/userflows/flows/UF-001-email-signup.md |
| UF-002 | Google Signup | Guest | Alternate Flow | UF-001 | Not Started | docs/central/userflows/flows/UF-002-google-signup.md |
| UF-003 | Login Recovery | Guest | Exception Flow | UF-004 | Not Started | docs/central/userflows/flows/UF-003-login-recovery.md |
| UF-004 | Email Login | Guest | Main Flow | none | Not Started | docs/central/userflows/flows/UF-004-email-login.md |
| UF-005 | Authenticated Conversation | Authenticated User | Main Flow | none | Not Started | docs/central/userflows/flows/UF-005-authenticated-conversation.md |
| UF-006 | Previous Conversation Continuation | Authenticated User | Subflow | UF-005 | Not Started | docs/central/userflows/flows/UF-006-previous-conversation-continuation.md |
