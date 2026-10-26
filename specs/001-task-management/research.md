# Research: Taskify Team Productivity

## Decisions

### .NET Aspire and .NET 8

- **Decision**: Use the .NET Aspire 13.6 release line for the AppHost, targeting the AppHost at .NET 10. Keep the Taskify Web, API, and Notifications services on .NET 8. Before implementation, verify the latest patch against the Aspire support policy and NuGet, then pin one consistent package set centrally.
- **Rationale**: Current Aspire support requires the .NET 10 SDK and a `net10.0` AppHost. The product services retain the constitution's .NET 8 target. This requires both SDKs locally and in CI. .NET 8 support ends in November 2026, so this is suitable only as the requested short-term hackathon target unless the runtime target is revisited.
- **Alternatives considered**: Aspire 9.5 on .NET 8 would keep every project on one target but is out of support. Targeting all services at .NET 10 would contradict the current project stack constraint. At research time, the Aspire support page listed 13.6.2 while NuGet listed a 13.6.4 PostgreSQL hosting package; therefore, do not assume either listing alone is authoritative for all Aspire packages.

### PostgreSQL Ownership and Aspire Wiring

- **Decision**: Declare PostgreSQL and the `taskify` database in AppHost. Give only Taskify API the database reference and data ownership. Use the Aspire Npgsql integration and EF Core's Npgsql provider in that service. Keep a durable local volume and pin the PostgreSQL image version.
- **Rationale**: A single service owner prevents UI and Notifications from bypassing the API's validation or coupling directly to its schema. Aspire references provide connection configuration and service discovery without fixed local ports.
- **Alternatives considered**: Sharing the database across services was rejected because it weakens service boundaries. Reading Aspire-provided connection settings directly was rejected in favor of the supported Npgsql integration for dependency injection, health checks, and telemetry.

### Board Interaction and Live Updates

- **Decision**: Implement browser drag-and-drop with the standard HTML Drag and Drop API through a small JavaScript interop module. Send task ID and target status to the Blazor Server component; let Blazor own rendered DOM updates. Provide keyboard-accessible move controls as an alternative. Use a distinct application SignalR hub for project-scoped board events; it is separate from Blazor Server's circuit transport.
- **Rationale**: Browser drag-and-drop has no need for a broad UI dependency, while accessible controls keep status movement available without a pointer. A dedicated application hub allows events to reach all open boards.
- **Alternatives considered**: Directly mutating Blazor-managed DOM was rejected because it can desynchronize rendering. Treating the Blazor circuit as the notifications contract was rejected because it is not an application-level broadcast API.

### Minimal Service Boundaries

- **Decision**: Use three runtime services: Taskify.Web (Blazor Server), Taskify.Api (REST and sole owner of projects, tasks, participants, comments, and PostgreSQL), and Taskify.Notifications (stateless REST event receiver and SignalR broadcaster). Taskify.Api publishes validated events to Notifications after database commits. Notifications checks project existence through Taskify.Api before accepting a project-group subscription; no service accesses another service's database.
- **Rationale**: This satisfies the requested REST API, microservice boundaries, and real-time board updates with minimal infrastructure. Notification outages do not undo committed task mutations. The UI reloads canonical board state from REST after connecting or reconnecting.
- **Alternatives considered**: A message broker and transactional outbox were not selected because durable event delivery is not a stated hackathon requirement and would add infrastructure. Direct client-to-database access was rejected.
- **Delivery limitation**: REST publication after commit is best-effort; a notification may be missed if the Notifications service is unavailable. Reconnection reloads canonical state. Add an outbox only if reliable event delivery becomes a requirement.

### Package and Verification Policy

- **Decision**: Use Microsoft Aspire hosting and service-discovery packages, ASP.NET Core and SignalR, EF Core, the Npgsql EF Core provider, Serilog, FluentValidation, and xUnit/Moq/FluentAssertions. Keep JavaScript limited to browser drag-and-drop interop. Validate every external REST and hub input at the receiving boundary.
- **Rationale**: These are the smallest dependencies that fulfill the chosen stack and the constitution's validation, documentation, and test requirements. Npgsql is essential for PostgreSQL support.
- **Alternatives considered**: Third-party drag-and-drop frameworks, a notification broker, and a general-purpose API gateway were rejected as unnecessary for the first phase.

## References

- [Aspire support policy](https://aspire.dev/support/)
- [Aspire 13 release notes](https://aspire.dev/whats-new/aspire-13/)
- [Aspire PostgreSQL hosting](https://aspire.dev/integrations/databases/postgres/postgres-host/)
- [Aspire PostgreSQL client connection](https://aspire.dev/integrations/databases/postgres/postgres-connect/)
- [Aspire service discovery](https://aspire.dev/fundamentals/service-discovery/)
- [Blazor event handling and drag-and-drop](https://learn.microsoft.com/en-us/aspnet/core/blazor/components/event-handling?view=aspnetcore-8.0)
- [Blazor JavaScript interop](https://learn.microsoft.com/en-us/aspnet/core/blazor/javascript-interoperability/call-javascript-from-dotnet?view=aspnetcore-8.0)
- [SignalR hubs](https://learn.microsoft.com/en-us/aspnet/core/signalr/hubs?view=aspnetcore-8.0)
- [SignalR hub context](https://learn.microsoft.com/en-us/aspnet/core/signalr/hubcontext?view=aspnetcore-8.0)
- [Microservice data ownership](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/architect-microservice-container-applications/data-sovereignty-per-microservice)
- [Identifying microservice boundaries](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/architect-microservice-container-applications/identify-microservice-domain-model-boundaries)
- [.NET release support policy](https://learn.microsoft.com/en-us/dotnet/core/releases-and-support)