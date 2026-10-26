# Quickstart: Taskify Validation

This guide validates the planned distributed feature after implementation. It does not provide implementation code.

## Prerequisites

- .NET 8 runtime and SDK for the service projects.
- .NET 10 SDK for the Aspire 13.6 AppHost.
- Aspire CLI compatible with the pinned Aspire package release.
- Docker Desktop or another supported Docker-compatible runtime for PostgreSQL.
- A modern browser for Blazor Server and drag-and-drop checks.
- Local demo configuration in `.env` or `appsettings.json`; never commit secrets.

## Build and Test

From the repository root:

```powershell
dotnet restore Taskify.sln
dotnet build Taskify.sln --no-restore
dotnet test Taskify.sln --no-build
```

Expected: all unit and integration tests pass. Integration tests start PostgreSQL and required services through Aspire; browser tests exercise core board interactions.

## Run the Distributed App

```powershell
dotnet run --project src/Taskify.AppHost/Taskify.AppHost.csproj
```

Expected: Aspire starts PostgreSQL, Taskify.Api, Taskify.Notifications, and Taskify.Web. Use the dashboard endpoint printed by Aspire to inspect resource health and open the Web app.

## Manual Acceptance Scenarios

1. On first launch, verify the roster has one Product Manager and four Engineers, and exactly three sample projects appear. Restart the app and verify they are not duplicated.
2. Select a profile and open a project. Verify the four board columns are visible and the selected profile is shown as the actor.
3. Create a project and task, assign an engineer, and verify blank names/titles and non-engineer assignees are rejected without partial changes.
4. Move a task between every status, including backward transitions. Verify an invalid status is rejected.
5. Open the same project in two browser sessions. Change assignment or status in one session and verify the other receives the update within two seconds in at least 95% of repeated checks.
6. Add a comment and verify its text, selected profile attribution, and server timestamp appear on the task.
7. Disconnect one board, make a task change in the other session, then reconnect. Verify the disconnected board reloads the latest saved state even though it missed the transient event.
8. Submit unknown project, task, actor, and assignee IDs through REST and verify clear ProblemDetails responses with no persisted changes.
9. Submit a stale task revision from two sessions. Verify the stale update returns `412 Precondition Failed` and the UI reloads the latest task before retrying.

## Contract References

- [REST/OpenAPI contract](contracts/openapi.yaml)
- [SignalR board update contract](contracts/signalr.md)
- [Entity model and validation rules](data-model.md)