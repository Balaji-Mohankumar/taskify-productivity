# Feature Specification: Taskify Team Productivity

**Feature Branch**: `001-task-management` (not created; no branch hook is configured)

**Created**: 2026-10-26

**Status**: Draft

**Input**: User description: "Develop Taskify, a team productivity platform where predefined users create projects, assign tasks, comment, and move tasks across Kanban columns (To Do, In Progress, In Review, Done). Five users (one product manager, four engineers), three sample projects, no login for this first phase."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Open a Project Board (Priority: P1)

As one of the predefined Taskify participants, I can identify which roster profile I am acting as and view a project board, so I can understand the current work and its progress.

**Why this priority**: The roster and project boards are the foundation for every collaboration workflow.

**Independent Test**: Start with a fresh Taskify workspace, choose a roster profile, and verify that the sample projects and their task columns are available.

**Acceptance Scenarios**:

1. **Given** a fresh workspace, **When** a participant opens Taskify, **Then** the roster contains one product manager and four engineers, and three sample projects are available.
2. **Given** the roster is displayed, **When** a participant chooses a profile, **Then** that profile is shown as the acting participant for subsequent actions without requesting login credentials.
3. **Given** a project is selected, **When** its board is opened, **Then** tasks are grouped under To Do, In Progress, In Review, and Done.

---

### User Story 2 - Create Projects and Assign Tasks (Priority: P1)

As a predefined participant, I can create a project, add tasks, and assign tasks to one of the engineers, so the team can organize and distribute work.

**Why this priority**: Creating and assigning work establishes the team's shared plan.

**Independent Test**: As a selected participant, create a uniquely named project, add a task, assign an engineer, and verify the task appears on that project's board with the selected assignee.

**Acceptance Scenarios**:

1. **Given** a selected roster profile, **When** the participant submits a project with a non-empty name, **Then** the project is created and can be opened.
2. **Given** an open project, **When** the participant creates a task with a non-empty title and selects an engineer, **Then** the task appears in To Do with that engineer as its assignee.
3. **Given** an existing task, **When** the participant changes its assignee to another rostered engineer, **Then** the board shows the new assignee.
4. **Given** an empty or whitespace-only project name or task title, **When** the participant submits it, **Then** the value is rejected and no project or task is created.

---

### User Story 3 - Collaborate on Task Progress (Priority: P2)

As a predefined participant, I can move a task between board columns and add comments, so the team can see progress and share context.

**Why this priority**: Status updates and comments make the board useful for ongoing team coordination after work is created.

**Independent Test**: Open an existing task, move it to another column, add a comment, and verify the status and attributed comment are visible after reopening the project.

**Acceptance Scenarios**:

1. **Given** a task in any board column, **When** a participant moves it to another column, **Then** it appears in the selected column and no longer appears in its previous column.
2. **Given** an existing task and a selected roster profile, **When** the participant submits a non-empty comment, **Then** the comment appears on that task with the selected profile and submission time.
3. **Given** a task and an empty or whitespace-only comment, **When** the participant submits it, **Then** the comment is rejected and the task's existing comments remain unchanged.
4. **Given** a task with a status change or comment, **When** the project is reopened, **Then** the latest status and comment remain available.
5. **Given** two participants are viewing the same project, **When** one participant creates or changes a task, assignment, status, or comment, **Then** the other open board reflects the change within two seconds without a manual refresh.
6. **Given** a participant reconnects to a project board, **When** the board becomes available, **Then** it displays the latest saved project and task state even if a notification was missed while disconnected.

### Edge Cases

- Reopening Taskify after initial setup does not create duplicate roster profiles or duplicate sample projects.
- A task cannot be assigned to a profile that is not one of the four predefined engineers.
- A task cannot be placed in a column outside To Do, In Progress, In Review, or Done.
- A project, task, engineer, or acting profile reference that does not exist is rejected without changing saved data.
- Empty or whitespace-only project names, task titles, and comments are rejected without partially applying the requested action.
- Any move between the four defined columns is allowed, including moving a task back to an earlier column.
- If a notification cannot be delivered, the underlying project or task change remains saved and is visible after the board refreshes or reconnects.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: On first use, the system MUST provide exactly five predefined profiles: one product manager and four engineers, and three sample projects.
- **FR-002**: For this first phase, the system MUST allow participants to select one of the predefined profiles without login credentials. The selected profile MUST be displayed as the acting participant for project, task, and comment actions.
- **FR-003**: The system MUST show each project's tasks grouped into exactly these columns: To Do, In Progress, In Review, and Done.
- **FR-004**: Any predefined participant MUST be able to create a project with a non-empty name and view that project.
- **FR-005**: Any predefined participant MUST be able to create a task in a project with a non-empty title and assign or reassign it to one of the four predefined engineers. A task MAY remain unassigned.
- **FR-006**: Any predefined participant MUST be able to move a task between any of the four defined columns.
- **FR-007**: Any predefined participant MUST be able to add a non-empty comment to a task. Each comment MUST show the selected acting profile and its submission time.
- **FR-008**: The system MUST retain created projects, tasks, assignments, status changes, and comments when a participant leaves and reopens the workspace.
- **FR-009**: The system MUST validate project names, task titles, comments, status values, and project, task, engineer, and profile references before accepting changes. Invalid values MUST be rejected with a clear explanation and MUST NOT partially change saved data.
- **FR-010**: The system MUST NOT provide sign-in, credential management, account registration, or roster editing in this first phase.
- **FR-011**: The system MUST notify participants viewing a project when a project or task is created, a task assignment or status changes, or a comment is added, so connected boards reflect the change without manual refresh.
- **FR-012**: The system MUST load the latest saved project and task state when a participant opens or reconnects to a board; a missed notification MUST NOT prevent recovery of current state.

### Key Entities *(include if feature involves data)*

- **Participant**: One of five predefined profiles, identified by a display name and a role of product manager or engineer. Engineers can be selected as task assignees.
- **Project**: A named work area that contains tasks and has one of the three initial sample projects as seed data.
- **Task**: A unit of project work with a title, optional description, optional engineer assignee, one of four defined statuses, and a record of its comments.
- **Comment**: Text associated with one task, attributed to a predefined participant and submission time.
- **Board notification**: A transient signal that a project or task changed. It prompts open boards to display current saved state and is not the authoritative record of that state.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: On a fresh workspace, all five predefined profiles and all three sample projects are available without participant setup.
- **SC-002**: In acceptance testing, 100% of valid project, task, assignment, status, and comment changes remain visible after the workspace is reopened.
- **SC-003**: In a usability walkthrough, at least four of five test participants can create a task, assign it to an engineer, move it to another column, and add a comment in three minutes or less without assistance.
- **SC-004**: In validation testing, 100% of empty required text values, invalid statuses, and unknown profile or project references are rejected without changing existing board data.
- **SC-005**: In a five-participant walkthrough on a normal local/demo network, at least 95% of project boards that are open during a change display that change within two seconds without manual refresh.

## Assumptions

- The five profiles can use generic display names such as Product Manager and Engineer 1 through Engineer 4; personal names were not provided.
- The three sample projects are available on first use. Participants may create additional projects, and the sample projects are not a limit on the number of projects.
- The product manager and engineers have the same project and task actions in this phase; role-specific permissions were not requested.
- Selecting a roster profile is a demo-only attribution mechanism, not authentication or proof of identity. Omitting login is an explicit first-phase exception to the constitution's authentication principle, based on the requested hackathon scope. Only synthetic, non-sensitive data may be used, and this exception must be reviewed before the feature is exposed outside a controlled demo.
- All four status transitions are allowed; task deletion, roster editing, and external integrations are outside this phase.
- Notifications are transient and best-effort in this phase; reopening or reconnecting to a board reloads authoritative saved state.