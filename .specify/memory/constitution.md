<!--
Sync Impact Report
Version change: unversioned -> 1.0.0 (initial versioned constitution)
Modified principles: Security-First Principle -> Security-First Principle (*TOBeNamedApp*)
Added sections: Scope; Development Workflow; Governance
Removed sections: Taskify-specific Project Concept & Overview
Follow-up TODOs: TODO(RATIFICATION_DATE): original adoption date was not recorded.
-->
# Project Constitution

## Core Principles

1. Tech Stack & Architecture: Use .NET Core 8, C#, ASP.NET Core Web API, Entity Framework Core, and a microservices architecture. Prefer modular design with clear service boundaries and gRPC/HTTP communication.
2. Security-First Principle (*TOBeNamedApp*): All user inputs must be strictly validated at the API boundary using FluentValidation or Data Annotations. Implement secure authentication, authorization, and zero trust on unvalidated inputs.
3. Dependency Discipline: Use only official Microsoft NuGet packages, Entity Framework Core, Serilog, FluentValidation, and essential enterprise libraries. Avoid bloated third-party packages.
4. Configuration & Secrets: Load all connection strings, secrets, and environment configurations strictly from local .env files or appsettings.json. Never hardcode secrets.
5. Code Quality & Documentation: Code must be fully documented with XML documentation comments on public APIs, clean architecture patterns, explicit type safety, and comprehensive error handling middleware.
6. Verification: Every microservice and feature must include unit and integration tests using xUnit, Moq, and FluentAssertions.

## Scope

These principles apply to every microservice, API, data-access component, configuration source, and test in the project.

## Development Workflow

Review each microservice and feature against these principles, including its required unit and integration tests, before considering the work complete.

## Governance

This constitution governs project architecture and engineering practices. Amendments require team approval, a documented rationale, and a version update following semantic versioning: MAJOR for incompatible governance changes, MINOR for new or materially expanded principles, and PATCH for clarifications. Reviews must check changes for compliance; any exception must be documented with its rationale.

**Version**: 1.0.0 | **Ratified**: TODO(RATIFICATION_DATE): original adoption date unknown | **Last Amended**: 2026-10-26
