# Data Model: Taskify Team Productivity

## Persistence Boundary

Taskify.Api is the sole owner of the PostgreSQL database and all entities below. Taskify.Web and Taskify.Notifications access task/project state only through Taskify.Api REST endpoints. Notifications are transient messages and are not persisted in this phase.

## Entities

### Participant

Represents one predefined demo profile.

| Field | Type | Rules |
|---|---|---|
| Id | UUID | Stable seeded identifier; primary key |
| DisplayName | string | Required; unique among the five seeded profiles |
| Role | enum | `ProductManager` or `Engineer` |
| IsSeeded | boolean | True for all five fixed profiles; roster editing is out of scope |

The initial seed contains exactly one product manager and four engineers. The acting participant ID is supplied with writes for attribution and reference validation; it is not authentication.

### Project

Represents a task board and its tasks.

| Field | Type | Rules |
|---|---|---|
| Id | UUID | Stable identifier; primary key |
| Name | string | Required, non-whitespace |
| CreatedAt | UTC timestamp | Server assigned |
| IsSample | boolean | True for the three initial sample projects |
| SeedKey | string, nullable | Unique when present; makes sample seeding idempotent |

Project names are not globally unique unless the implementation adds an explicitly tested requirement; generated IDs establish identity.

### Task

Represents work on a project board.

| Field | Type | Rules |
|---|---|---|
| Id | UUID | Primary key |
| ProjectId | UUID | Required foreign key to Project |
| Title | string | Required, non-whitespace |
| Description | string, nullable | Optional |
| AssigneeId | UUID, nullable | If present, must reference a Participant with role Engineer |
| Status | enum | Exactly `ToDo`, `InProgress`, `InReview`, or `Done` |
| Revision | integer | Server-managed optimistic concurrency revision |
| CreatedAt | UTC timestamp | Server assigned |
| UpdatedAt | UTC timestamp | Server updated |

Any of the four status transitions are valid, including backward moves. No task deletion is included in this phase.

### Comment

Represents an attributed comment on a task.

| Field | Type | Rules |
|---|---|---|
| Id | UUID | Primary key |
| TaskId | UUID | Required foreign key to Task |
| AuthorId | UUID | Required foreign key to Participant |
| Body | string | Required, non-whitespace |
| CreatedAt | UTC timestamp | Server assigned |

Comments are append-only in this phase; editing and deletion are out of scope.

### BoardChanged Notification (transient contract)

Represents a signal that saved project state changed. It is not an authoritative copy of the entity and is not stored.

| Field | Type | Rules |
|---|---|---|
| EventId | UUID | Unique per emitted event |
| ProjectId | UUID | Required; determines SignalR group |
| TaskId | UUID, nullable | Present when a specific task changed |
| ChangeType | enum | `ProjectCreated`, `TaskCreated`, `TaskUpdated`, or `CommentAdded` |
| ActorId | UUID, nullable | Predefined profile attribution, not verified identity |
| OccurredAt | UTC timestamp | Server assigned |

If a notification is lost, the client reloads project/task data from REST on reconnect.

## Relationships

- Project has zero or more Tasks.
- Task has zero or one Engineer Participant as assignee.
- Task has zero or more Comments.
- Comment has exactly one Participant author.
- Project and Participant sample records are inserted with stable IDs and seed keys so repeated startup does not duplicate them.

## Validation and Integrity

- Reject missing, empty, or whitespace-only project names, task titles, and comment bodies.
- Validate project, task, actor, and assignee references at the API boundary before mutation.
- Reject assignees that do not exist or whose role is not Engineer.
- Reject status values outside the four defined enum values.
- Keep changes atomic: invalid input or failed database validation must not partially persist.
- Require the expected task revision on updates. Return `412 Precondition Failed` when it is stale and include the current revision so the UI can reload.
- Enforce foreign keys and the unique seeded-profile display names/seed keys in PostgreSQL.