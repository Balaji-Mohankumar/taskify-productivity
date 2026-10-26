# SignalR Contract: Taskify Board Updates

## Endpoint

- Hub path: `/hubs/boards`
- Transport: SignalR over the browser's supported WebSocket/HTTP transports.
- Hub events are transient. REST remains the source of truth.

## Client-to-Server Methods

### `JoinProject(projectId, participantId)`

Join the project-specific board group after validating that the project and selected predefined participant exist through Taskify.Api. The participant ID is demo attribution, not authentication or authorization. Any participant may view a project in this phase.

### `LeaveProject(projectId)`

Leave the project-specific board group.

## Server-to-Client Events

### `BoardChanged(event)`

Payload fields: `eventId`, `projectId`, optional `taskId`, `changeType`, optional `actorId`, and server-assigned `occurredAt`. `changeType` is one of `ProjectCreated`, `TaskCreated`, `TaskUpdated`, or `CommentAdded`.

The event indicates that authoritative state changed; it does not carry the full task or project. The client reloads affected project/task data through REST. On connect or reconnect, the client reloads the full board before treating it as current.

## Delivery and Failure Behavior

- Taskify.Api commits business data before posting the event to Notifications.
- Notifications validates the internal service credential and payload before broadcasting to the project group.
- Event publication is best-effort. Failure does not roll back an already committed API change.
- A disconnected client may miss an event; reconnect triggers a REST reload.
- Hub inputs and project-group membership are validated. No profile ID or group membership is treated as identity proof.