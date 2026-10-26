# Implementation Plan: Taskify Team Productivity

**Branch**: `001-task-management` | **Date**: 2026-10-27 | **Spec**: [spec.md](spec.md)

**Input**: Feature specification from `/specs/001-task-management/spec.md`; planning choices: .NET Aspire, PostgreSQL, Blazor Server with drag-and-drop and live updates, and REST APIs for projects, tasks, and notifications.

## Summary

Build a small distributed Taskify application with a Blazor Server board, a REST API that exclusively owns project/task data in PostgreSQL, and a stateless Notifications service that broadcasts project-scoped SignalR events. Use Aspire for local orchestration and service discovery. Persist every business change through the API; notifications are best-effort signals, with REST reload on reconnect as the recovery path.

## Technical Context

**Language/Version**: C#; Web, API, and Notifications target .NET 8. The Aspire AppHost targets .NET 10 because the supported Aspire 13.6 line requires the .NET 10 SDK.

**Primary Dependencies**: ASP.NET Core Web API, Blazor Server, .NET Aspire 13.6.x, EF Core, Npgsql EF Core provider, Aspire PostgreSQL and service-discovery integrations, SignalR, FluentValidation, Serilog, xUnit, Moq, FluentAssertions, and Microsoft Playwright for browser-level integration checks.

**Storage**: PostgreSQL, owned exclusively by Taskify.Api. Participants and three initial sample projects are inserted idempotently. Notifications are transient and are not stored in a separate database.

**Testing**: xUnit unit and integration tests for Web, API, and Notifications; Moq and FluentAssertions; Aspire.Hosting.Testing to start the real PostgreSQL-backed distributed app; Microsoft Playwright for core board flows.

**Target Platform**: Local hackathon/demo environment on Windows, macOS, or Linux with Docker-compatible container runtime; browser-based clients.

**Project Type**: Distributed web application with a Blazor Server UI and two backend services.

**Performance Goals**: At least 95% of connected boards show a committed task/project change within 2 seconds on a normal local/demo network, matching SC-005. Support the five predefined participants and three seeded projects without manual setup.

**Constraints**: No participant login in this phase; profile selection is attribution only, not identity verification. Use synthetic, non-sensitive data and keep the app in a controlled demo environment. Validate all API and hub inputs. Load secrets and configuration only from local `.env` files or `appsettings.json`. Use a centrally pinned compatible Aspire 13.6.x package set, verified against current support information before implementation. Keep PostgreSQL versions pinned when using a persistent volume.

**Scale/Scope**: Five predefined profiles, three seeded projects, a small hackathon team, and additional projects/tasks created during the demo. No production availability or high-volume target is claimed.

## Constitution Check

*Gate before research: evaluated. Re-evaluated after design below.*

| Principle | Design evidence | Gate |
|---|---|---|
| .NET 8, C#, ASP.NET Core, EF Core, microservices | Web, API, Notifications target .NET 8; API and Notifications have separate boundaries; PostgreSQL access is isolated to API. | Pass with bounded AppHost tooling exception below |
| Security-first input validation and authentication | FluentValidation at REST boundaries; validate entity references and hub group subscriptions. No-login is limited to the explicitly requested controlled demo; selected profile is not authentication. | Pass for demo scope; no external exposure |
| Dependency discipline | Official Aspire and Microsoft packages, EF Core, Npgsql provider, Serilog, FluentValidation, and required testing packages only. No broker or general API gateway. | Pass |
| Configuration and secrets | Aspire-injected service configuration; secrets from local `.env`/`appsettings.json`, excluded from source control; no hardcoded credentials. | Pass |
| Documentation and error handling | XML docs for public REST and hub contracts; ProblemDetails middleware; explicit request/response types and validation errors. | Pass |
| Verification | Unit and integration tests for every runtime service and feature using xUnit, Moq, and FluentAssertions; browser coverage for drag/drop and live updates. | Pass |

**Bounded exception**: Supported Aspire 13.6 requires the .NET 10 SDK and a `net10.0` AppHost. The AppHost is orchestration tooling, not a runtime microservice; Web, API, and Notifications remain .NET 8. The latest compatible Aspire patch must be verified and pinned before implementation. This exception and rationale are documented for project review.

**Security boundary**: There is no user authentication in this first phase, as the feature spec explicitly requires. Only synthetic data is permitted. The Task API validates the selected participant ID but cannot establish the caller's identity. The internal notification-ingress endpoint uses a service-to-service secret from configuration; the secret must never be exposed to the browser. Do not expose this demo publicly.

## Project Structure

### Documentation (this feature)

```text
specs/001-task-management/
├── plan.md
├── research.md
├── data-model.md
├── contracts/
│   ├── openapi.yaml
│   └── signalr.md
├── quickstart.md
└── tasks.md             # Created later by /speckit-tasks
```

### Source Code (repository root)

```text
Taskify.sln
src/
├── Taskify.AppHost/                 # .NET 10 Aspire orchestration only
├── Taskify.ServiceDefaults/         # Shared health, telemetry, discovery defaults
├── Taskify.Web/                     # .NET 8 Blazor Server UI and drag/drop interop
├── Taskify.Api/                     # .NET 8 REST API and sole PostgreSQL owner
│   ├── Api/                          # Endpoints, validation, ProblemDetails
│   ├── Application/                  # Use cases and service contracts
│   ├── Domain/                       # Project, Task, Comment, Participant
│   └── Infrastructure/               # EF Core persistence and notification client
└── Taskify.Notifications/           # .NET 8 internal REST ingress and SignalR hub
tests/
├── Taskify.Web.UnitTests/
├── Taskify.Api.UnitTests/
├── Taskify.Notifications.UnitTests/
├── Taskify.IntegrationTests/        # Aspire-hosted service and PostgreSQL checks
└── Taskify.BrowserTests/            # Microsoft Playwright user-flow coverage
```

**Structure Decision**: Keep the Web UI, data-owning REST API, and notification broadcaster in distinct deployable projects. Only Taskify.Api references PostgreSQL. Do not share domain entities across services; share only hosting defaults. This keeps service ownership clear while avoiding extra services, gateways, or messaging infrastructure.

## Design Decisions

- The AppHost declares PostgreSQL and the three services. Only the API receives a database reference. Aspire service discovery supplies service URLs; no fixed local ports are embedded in clients.
- REST owns authoritative reads and writes. Mutations commit to PostgreSQL before the API posts a notification event. The notification post is best-effort; the UI reloads the full board from REST after connect/reconnect.
- The Notifications service accepts internal events only from Taskify.Api using a configured service secret. It validates event shape and publishes to a project group. Hub subscription validates that the project exists; group membership is not an authorization mechanism.
- The selected roster profile lives in Blazor scoped state and is sent as an actor identifier on writes. It is attribution only and is not a trusted identity claim.
- Use an integer task revision for optimistic concurrency. A stale task update returns `412 Precondition Failed`; the UI reloads the task and asks the participant to retry rather than silently overwriting another change.
- Seed the five stable participant IDs and three stable sample-project IDs idempotently. Use generic display names and demo project names because names were not supplied.
- Use HTML drag-and-drop through a small JavaScript interop module. JavaScript reports the drop; Blazor remains the owner of rendered DOM. Provide keyboard-accessible status controls for the same transitions.
- Use EF Core migrations for schema changes and ProblemDetails for consistent REST errors. Public endpoints and hub methods receive XML documentation.

## Complexity Tracking

| Exception or added component | Why needed | Simpler alternative rejected because |
|---|---|---|
| .NET 10 Aspire AppHost alongside .NET 8 services | Supported Aspire 13.6 requires a .NET 10 AppHost; services preserve the constitution's .NET 8 target. | Aspire 9.5 is out of support; moving all runtime services to .NET 10 is unnecessary for orchestration and changes the stated target. |
| Separate Notifications service | User explicitly requested live board updates and REST notifications; a stateless broadcaster keeps notification delivery separate from task data ownership. | A hub inside the API would be smaller, but would not preserve the requested notification service boundary. |
| Best-effort HTTP event post, no broker/outbox | Five-user hackathon scope can recover missed events by reloading REST state. | A broker and transactional outbox add infrastructure without a stated durable-delivery requirement. |
