---
description: "Task list for Taskify Team Productivity implementation"
---

# Tasks: Taskify Team Productivity

**Input**: Design documents from `/specs/001-task-management/`

**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/, quickstart.md

**Tests**: Required by the project constitution. Each runtime service and feature receives unit and integration coverage using xUnit, Moq, and FluentAssertions; browser flows use Microsoft Playwright.

**Organization**: Tasks are grouped by user story for incremental implementation and validation.

## Format: `[ID] [P?] [Story?] Description with exact path`

- **[P]**: The task can run in parallel with another task because it changes different files and has no dependency on unfinished work.
- **[Story]**: User story label (`US1`, `US2`, or `US3`) for story-phase tasks.
- Every task includes a concrete repository path.

## Phase 1: Setup

**Purpose**: Create the solution and shared build configuration described in plan.md.

- [ ] T001 Create `Taskify.sln` and the AppHost, ServiceDefaults, Web, API, Notifications, unit-test, integration-test, and browser-test projects at the exact paths listed in `specs/001-task-management/plan.md`.
- [ ] T002 Configure `Directory.Build.props`, `Directory.Packages.props`, `global.json`, and `.gitignore` for nullable C#, XML documentation generation, central package management, .NET 8 runtime projects, the .NET 10 AppHost, and ignored local secrets and build outputs; verify and pin a compatible supported Aspire 13.6.x package set.
- [ ] T003 Define PostgreSQL and the Web, API, and Notifications resources plus explicit service references in `src/Taskify.AppHost/Program.cs`; grant database access only to Taskify.Api.

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Complete shared service defaults, API boundary behavior, and integration-test infrastructure before user-story work.

- [ ] T004 [P] Implement shared health checks, telemetry, and Aspire service-discovery defaults in `src/Taskify.ServiceDefaults/Extensions.cs` and `src/Taskify.ServiceDefaults/Taskify.ServiceDefaults.csproj`.
- [ ] T005 [P] Configure Serilog, FluentValidation registration, ProblemDetails responses, and exception-handling middleware in `src/Taskify.Api/Program.cs` and `src/Taskify.Api/Api/ApiExceptionHandlingExtensions.cs`.
- [ ] T006 Configure the API's EF Core/Npgsql connection, `TaskifyDbContext`, and migration assembly in `src/Taskify.Api/Infrastructure/TaskifyDbContext.cs` and `src/Taskify.Api/Program.cs`; read the database connection only from Aspire-injected configuration or approved local settings.
- [ ] T007 Configure the Web-to-API typed HTTP client with Aspire service discovery in `src/Taskify.Web/Program.cs` and `src/Taskify.Web/Services/TaskifyApiClient.cs`.
- [ ] T008 Create a reusable Aspire integration-test fixture that starts the AppHost and PostgreSQL in `tests/Taskify.IntegrationTests/AspireFixture.cs`.
- [ ] T009 Add shared test utilities for FluentAssertions and Moq setup in `tests/Taskify.Api.UnitTests/Usings.cs`, `tests/Taskify.Web.UnitTests/Usings.cs`, and `tests/Taskify.Notifications.UnitTests/Usings.cs`.

**Checkpoint**: Solution builds; Aspire can start the database and service resources; API error handling and the shared integration fixture are available.

---

## Phase 3: User Story 1 - Open a Project Board (Priority: P1)

**Goal**: Let a participant select a predefined profile, find a project, and view its board with the four status columns and seeded project data.

**Independent Test**: Start from an empty database, load the roster and three sample projects, select a profile, open a project, and confirm the board presents To Do, In Progress, In Review, and Done without creating duplicate seed records on restart.

### Tests for User Story 1

- [ ] T010 [P] [US1] Add unit tests proving seed identities contain exactly one ProductManager and four Engineers and rerunning seeding does not duplicate records in `tests/Taskify.Api.UnitTests/ParticipantSeedTests.cs` and `tests/Taskify.Api.UnitTests/ProjectSeedTests.cs`.
- [ ] T011 [P] [US1] Add Aspire integration tests for participant retrieval, exactly three sample projects, project/task listing, and repeated-startup seed idempotency in `tests/Taskify.IntegrationTests/ProjectBoardApiTests.cs`.
- [ ] T012 [P] [US1] Add a Playwright browser test for profile selection, project listing, board columns, and empty project state in `tests/Taskify.BrowserTests/BoardNavigationTests.cs`.

### Implementation for User Story 1

- [ ] T013 [US1] Define the Participant entity with UUID primary key, required display name unique among the five seeded profiles, `ProductManager`/`Engineer` role, and `IsSeeded` flag in `src/Taskify.Api/Domain/Participant.cs`.
- [ ] T014 [US1] Define the Project entity with UUID primary key, required non-whitespace name, server-assigned UTC `CreatedAt`, `IsSample` flag, and nullable unique `SeedKey` in `src/Taskify.Api/Domain/Project.cs`.
- [ ] T015 [US1] Add Participant and Project EF Core mappings, foreign-key/index constraints, stable participant IDs, and three stable sample-project seed keys in `src/Taskify.Api/Infrastructure/TaskifyDbContext.cs` and `src/Taskify.Api/Infrastructure/SeedData.cs`; make seed application idempotent.
- [ ] T016 [US1] Create and apply the initial PostgreSQL migration for Participant and Project in `src/Taskify.Api/Infrastructure/Migrations/`.
- [ ] T017 [US1] Implement read-only participant and project queries in `src/Taskify.Api/Application/ParticipantQueries.cs` and `src/Taskify.Api/Application/ProjectQueries.cs`.
- [ ] T018 [US1] Implement `GET /api/participants`, `GET /api/projects`, `GET /api/projects/{projectId}`, and `GET /api/projects/{projectId}/tasks` in `src/Taskify.Api/Api/ParticipantsController.cs` and `src/Taskify.Api/Api/ProjectsController.cs`; return documented ProblemDetails for unknown project IDs.
- [ ] T019 [US1] Add `GET /api/participants`, participant response schema, and all project read responses to `specs/001-task-management/contracts/openapi.yaml` so profile selection and project navigation have complete contracts.
- [ ] T020 [US1] Implement the API client methods and scoped selected-participant state in `src/Taskify.Web/Services/TaskifyApiClient.cs` and `src/Taskify.Web/State/ParticipantSession.cs`; display selected profile as attribution, not authentication.
- [ ] T021 [US1] Create the profile selector and project list pages in `src/Taskify.Web/Components/Pages/Projects.razor` and `src/Taskify.Web/Components/Shared/ParticipantSelector.razor`.
- [ ] T022 [US1] Create the project board and four empty-capable columns in `src/Taskify.Web/Components/Pages/ProjectBoard.razor` and `src/Taskify.Web/Components/Shared/TaskColumn.razor`; use only the canonical To Do, In Progress, In Review, and Done statuses.
- [ ] T023 [US1] Add Web unit tests for selected-profile state and API client response mapping in `tests/Taskify.Web.UnitTests/ParticipantSessionTests.cs` and `tests/Taskify.Web.UnitTests/TaskifyApiClientTests.cs`.

**Checkpoint**: User Story 1 works independently with five profiles, three sample projects, and an empty board grouped into four columns.

---

## Phase 4: User Story 2 - Create Projects and Assign Tasks (Priority: P1)

**Goal**: Let predefined participants create projects and tasks and assign tasks to an engineer.

**Independent Test**: Create a project and task through the UI, assign one of the four engineers, and confirm the task and assignment persist after reopening; reject blank required fields and invalid assignees without partial writes.

### Tests for User Story 2

- [ ] T024 [P] [US2] Add unit tests for project-name, task-title, description, and engineer-assignee validation in `tests/Taskify.Api.UnitTests/ProjectValidationTests.cs` and `tests/Taskify.Api.UnitTests/TaskValidationTests.cs`.
- [ ] T025 [P] [US2] Add Aspire integration tests for project creation, task creation, unassigned tasks, engineer assignment/reassignment, invalid references, and persistence in `tests/Taskify.IntegrationTests/ProjectAndTaskWriteApiTests.cs`.
- [ ] T026 [P] [US2] Add a Playwright browser test for creating a project, creating a task, and assigning/reassigning an engineer in `tests/Taskify.BrowserTests/ProjectAndTaskCreationTests.cs`.

### Implementation for User Story 2

- [ ] T027 [US2] Align `specs/001-task-management/data-model.md` with the contract limits: project name required/non-whitespace with maximum 120 characters; task title required/non-whitespace with maximum 200 characters; optional description maximum 4000 characters; preserve the existing UUID, nullable-assignee, and engineer-only assignment rules.
- [ ] T028 [US2] Add FluentValidation rules for project name length 1-120 after trimming whitespace, task title length 1-200 after trimming whitespace, optional description maximum 4000 characters, and an assignee that is either null or an existing Engineer in `src/Taskify.Api/Application/Validation/CreateProjectValidator.cs` and `src/Taskify.Api/Application/Validation/CreateTaskValidator.cs`.
- [ ] T029 [US2] Define the Task entity with UUID primary key, required Project foreign key, required title, nullable description, nullable Engineer assignee, `ToDo` initial status, positive revision, and server-assigned UTC timestamps in `src/Taskify.Api/Domain/TaskItem.cs`.
- [ ] T030 [US2] Add Task EF Core mapping, Project/Participant foreign keys, four-status enum conversion, and indexes in `src/Taskify.Api/Infrastructure/TaskifyDbContext.cs`; add and apply the task schema migration in `src/Taskify.Api/Infrastructure/Migrations/`.
- [ ] T031 [US2] Implement project creation and task create/list use cases with atomic persistence and initial `ToDo` status in `src/Taskify.Api/Application/CreateProject.cs`, `src/Taskify.Api/Application/CreateTask.cs`, and `src/Taskify.Api/Application/TaskQueries.cs`.
- [ ] T032 [US2] Implement `POST /api/projects` and `POST /api/projects/{projectId}/tasks` with required actor attribution, validation, `201 Created` responses, and ProblemDetails errors in `src/Taskify.Api/Api/ProjectsController.cs` and `src/Taskify.Api/Api/TasksController.cs`.
- [ ] T033 [US2] Implement task assignment and reassignment updates that accept null or an existing Engineer ID, reject unknown/non-engineer IDs, and require the current task revision in `src/Taskify.Api/Application/UpdateTaskAssignment.cs` and `src/Taskify.Api/Api/TasksController.cs`.
- [ ] T034 [US2] Align create-project and create-task OpenAPI schemas with the data-model length constraints and actor-attribution headers in `specs/001-task-management/contracts/openapi.yaml`.
- [ ] T035 [US2] Add project creation, task creation, assignment selection, and field-level validation messages to `src/Taskify.Web/Components/Pages/Projects.razor`, `src/Taskify.Web/Components/Pages/ProjectBoard.razor`, and `src/Taskify.Web/Components/Shared/TaskEditor.razor`.
- [ ] T036 [US2] Add API unit tests for project/task validators and assignment rules, including blank/whitespace text, maximum lengths, null assignee, unknown participant, and non-engineer assignee in `tests/Taskify.Api.UnitTests/ProjectValidationTests.cs` and `tests/Taskify.Api.UnitTests/TaskValidationTests.cs`.

**Checkpoint**: User Story 2 works independently on a seeded project and persists created projects, tasks, and engineer assignments.

---

## Phase 5: User Story 3 - Collaborate on Task Progress (Priority: P2)

**Goal**: Let participants move tasks among all four statuses, add attributed comments, and see live project updates with reconnect recovery.

**Independent Test**: Open one project in two browser sessions, change task status/assignment and add a comment in one session, observe updates in the other, verify all backward and forward transitions, then disconnect/reconnect and confirm REST reload restores current saved state. Verify stale revisions return `412` without overwriting newer data.

### Tests for User Story 3

- [ ] T037 [P] [US3] Add unit tests for all four allowed status values, revision increments, stale-revision rejection, and comment body length 1-2000 after trimming whitespace in `tests/Taskify.Api.UnitTests/TaskStatusTests.cs` and `tests/Taskify.Api.UnitTests/CommentValidationTests.cs`.
- [ ] T038 [P] [US3] Add Notifications unit tests for invalid event payloads, invalid service credentials, project-group membership validation, and project-scoped broadcasts in `tests/Taskify.Notifications.UnitTests/NotificationIngressTests.cs` and `tests/Taskify.Notifications.UnitTests/BoardHubTests.cs`.
- [ ] T039 [P] [US3] Add Aspire integration tests for comment persistence, comment history retrieval, notification failure after commit, and stale task revisions returning `412` in `tests/Taskify.IntegrationTests/TaskCollaborationApiTests.cs`.
- [ ] T040 [P] [US3] Add a Playwright multi-session test for status change, assignment/comment live updates within two seconds, keyboard status controls, and reconnect reload in `tests/Taskify.BrowserTests/LiveBoardCollaborationTests.cs`.

### Implementation for User Story 3

- [ ] T041 [US3] Define the append-only Comment entity with UUID primary key, required Task and Participant foreign keys, required non-whitespace Body, and server-assigned UTC CreatedAt in `src/Taskify.Api/Domain/TaskComment.cs`.
- [ ] T042 [US3] Add Comment EF Core mapping and Task/Participant foreign keys, then create and apply the comment schema migration in `src/Taskify.Api/Infrastructure/TaskifyDbContext.cs` and `src/Taskify.Api/Infrastructure/Migrations/`.
- [ ] T043 [US3] Implement append-only comment creation and ordered comment-history queries in `src/Taskify.Api/Application/AddTaskComment.cs` and `src/Taskify.Api/Application/TaskCommentQueries.cs`; enforce comment length 1-2000 after trimming whitespace.
- [ ] T044 [US3] Implement `GET /api/tasks/{taskId}/comments` and `POST /api/tasks/{taskId}/comments` with actor-reference validation, server-assigned author/time, and documented ProblemDetails in `src/Taskify.Api/Api/TaskCommentsController.cs`.
- [ ] T045 [US3] Add comment-history response and retrieval errors to `specs/001-task-management/contracts/openapi.yaml` and confirm request/response constraints match `specs/001-task-management/data-model.md`.
- [ ] T046 [US3] Implement task-status updates for any transition among `ToDo`, `InProgress`, `InReview`, and `Done`, requiring `If-Match` and returning `412 Precondition Failed` with the current task revision on stale updates in `src/Taskify.Api/Application/UpdateTaskStatus.cs` and `src/Taskify.Api/Api/TasksController.cs`.
- [ ] T047 [US3] Implement the internal `POST /internal/notifications` endpoint with configured service-key validation, strict event validation, and `202 Accepted` responses in `src/Taskify.Notifications/Api/NotificationsController.cs`.
- [ ] T048 [US3] Implement the `/hubs/boards` SignalR hub with validated `JoinProject`/`LeaveProject` methods and project-scoped `BoardChanged` broadcasts in `src/Taskify.Notifications/Hubs/BoardHub.cs`.
- [ ] T049 [US3] Implement a typed Notifications client in `src/Taskify.Api/Infrastructure/NotificationsClient.cs` that sends `ProjectCreated`, `TaskCreated`, `TaskUpdated`, or `CommentAdded` only after database commit and treats delivery failure as best-effort without rolling back saved data.
- [ ] T050 [US3] Configure the internal service key through local configuration and Aspire service references without exposing it to the browser in `src/Taskify.AppHost/Program.cs`, `src/Taskify.Api/Program.cs`, and `src/Taskify.Notifications/Program.cs`.
- [ ] T051 [US3] Add the SignalR client, project join/leave lifecycle, canonical REST reload on connect/reconnect, and refresh-on-`BoardChanged` behavior in `src/Taskify.Web/Services/BoardNotificationClient.cs` and `src/Taskify.Web/Components/Pages/ProjectBoard.razor`.
- [ ] T052 [US3] Add the task comment history/composer and non-empty comment validation in `src/Taskify.Web/Components/Shared/TaskComments.razor` and `src/Taskify.Web/Components/Shared/TaskCard.razor`.
- [ ] T053 [US3] Add pointer drag-and-drop interop that reports task ID and target status without directly mutating Blazor-managed DOM in `src/Taskify.Web/Components/Shared/TaskColumn.razor.js` and `src/Taskify.Web/Components/Shared/TaskCard.razor`.
- [ ] T054 [US3] Add keyboard-accessible status controls for the same four task transitions in `src/Taskify.Web/Components/Shared/TaskCard.razor` and `src/Taskify.Web/Components/Shared/TaskStatusSelector.razor`.
- [ ] T055 [US3] Add integration coverage for Notifications service-to-service event ingress and SignalR group broadcast in `tests/Taskify.IntegrationTests/NotificationsIntegrationTests.cs`.

**Checkpoint**: User Story 3 delivers persisted comments/status, concurrency protection, live updates, keyboard/pointer interaction, and reconnect recovery.

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Close documentation, configuration, security, and end-to-end validation gaps across the completed stories.

- [ ] T056 [P] Add XML documentation comments to every public REST request/response type, controller action, hub method, and notification event in `src/Taskify.Api/`, `src/Taskify.Notifications/`, and `src/Taskify.Web/`.
- [ ] T057 [P] Add public API contract/error handling tests for ProblemDetails status, title, and field errors in `tests/Taskify.IntegrationTests/ApiProblemDetailsTests.cs`.
- [ ] T058 Review local secrets, service-key exposure, synthetic-data-only scope, and absence of public identity claims in `src/Taskify.AppHost/Program.cs`, `src/Taskify.Api/Program.cs`, `src/Taskify.Notifications/Program.cs`, and `specs/001-task-management/quickstart.md`.
- [ ] T059 Run every command and manual scenario in `specs/001-task-management/quickstart.md`; fix any failing acceptance criterion in the owning project or test path.
- [ ] T060 Record live-update timing results against the 95%-within-two-seconds target and document the tested local/demo conditions in `specs/001-task-management/quickstart.md`.

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies; creates the solution, package policy, and Aspire resource graph.
- **Foundational (Phase 2)**: Depends on Setup; blocks all user stories.
- **User Story 1 (Phase 3)**: Depends on Foundational; provides seeded roster/projects and the first independently demonstrable board.
- **User Story 2 (Phase 4)**: Depends on Foundational and the US1 board/client components for full UI integration. Its API validation and persistence tasks can be developed independently once Foundational is complete.
- **User Story 3 (Phase 5)**: Depends on the task persistence and board from US1/US2; adds comments, task status updates, notifications, and reconnect behavior.
- **Polish (Phase 6)**: Depends on all stories selected for the demo.

### User Story Dependencies

- **US1 (P1)**: Starts after Phase 2. No dependency on other user stories.
- **US2 (P1)**: API/entity work starts after Phase 2; board UI integration uses US1 project and client components.
- **US3 (P2)**: Requires task persistence and board components from US1/US2.

### Parallel Opportunities

- **Phase 2**: T004 and T005 can run in parallel after T001-T003 because they edit separate projects/files.
- **US1**: T010, T011, and T012 can be authored in parallel; T013 and T014 can be implemented in parallel; T020's independent Web unit tests can be authored alongside API query/endpoint work after the contracts are stable.
- **US2**: T024, T025, and T026 can be authored in parallel; project and task validators in T028 can be split across their separate files; API and browser test files can be developed independently.
- **US3**: T037-T040 can be authored in parallel; Comments, Notifications, and Web components can be implemented in parallel after their respective contracts are agreed and prior-story interfaces are stable.
- **Across stories**: US2 API work may overlap with US1 UI work after Phase 2, but US2 board integration must wait for the US1 board/client components. US3 starts after the task API and board surfaces exist.

### Parallel Example: User Story 1

After Phase 2 completes, author the US1 test files concurrently: `tests/Taskify.Api.UnitTests/ParticipantSeedTests.cs`, `tests/Taskify.IntegrationTests/ProjectBoardApiTests.cs`, and `tests/Taskify.BrowserTests/BoardNavigationTests.cs`. Participant and Project domain files can also be created independently, then integrated through the shared DbContext and seed setup.

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1 setup and Phase 2 foundations.
2. Complete US1: seed the five profiles and three projects; expose participant/project reads; allow profile selection and board viewing.
3. Run the US1 API integration, UI unit, and browser tests independently.
4. Demo the read-only board MVP before adding project/task writes.

### Incremental Delivery

1. Setup + Foundational: buildable Aspire solution, PostgreSQL ownership, validation/error handling, and test harness.
2. US1: seeded roster/projects and independently viewable boards.
3. US2: project/task creation and engineer assignment, persisted and independently tested.
4. US3: status transitions, comments, live updates, concurrency behavior, and reconnect recovery.
5. Polish: XML docs, security review, complete quickstart, and record live-update target results.

### Parallel Team Strategy

1. Complete Setup and Foundational phases together to stabilize project references and service contracts.
2. After foundations, one contributor can implement US1 API/seed work while another prepares US1 Web pages and tests against the agreed contract.
3. US2 API validation/persistence can proceed alongside US1 UI work; integrate US2 forms after the board/client surface is ready.
4. Once US2 APIs stabilize, separate contributors can work on API comments/status, Notifications, and Web real-time behavior using the contracts in `specs/001-task-management/contracts/`.