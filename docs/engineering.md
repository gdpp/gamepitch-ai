# Engineering Guide

| Field            | Value             |
| ---------------- | ----------------- |
| **Project**      | GamePitch AI      |
| **Document**     | Engineering Guide |
| **Version**      | 1.0               |
| **Status**       | Draft             |
| **Owner**        | Gustavo Perez     |
| **Last Updated** | 2026-07-01        |

# Table of Contents

1. Purpose
2. Engineering Principles
3. Development Workflow
4. Git Strategy
5. Coding Standards
6. Dependency Management
7. Docker Strategy
8. CI/CD
9. Observability
10. Feature Flags
11. Technical Documentation
12. Definition of Done

# 1. Purpose

The purpose of this document is to define the engineering practices adopted throughout the GamePitch AI project.

These guidelines aim to ensure consistency, maintainability, code quality, and long-term sustainability regardless of team size.

Engineering practices are considered part of the product itself rather than optional processes.

# 2. Engineering Principles

The following principles guide every engineering decision made throughout the project.

## Simplicity Over Complexity

Solutions should remain as simple as possible while satisfying current business requirements.

Additional complexity should only be introduced when justified by measurable value.

## Readability First

Code is read far more often than it is written.

Naming, structure, and organization should prioritize clarity over cleverness.

Every module should communicate its intent without requiring extensive documentation.

## Clean Code

The codebase should emphasize:

- Small functions
- Single Responsibility Principle
- Explicit naming
- Low coupling
- High cohesion
- Minimal side effects

Technical debt should be identified early and addressed continuously.

## Architecture Before Features

Every feature should align with the architectural decisions documented in `architecture.md`.

Business requirements should never justify violating architectural boundaries without explicit review.

## Incremental Delivery

Features should be delivered in small, reviewable increments.

Large, long-lived branches should be avoided.

Each change should produce a working and testable application.

## Automation First

Repetitive engineering tasks should be automated whenever possible.

Examples include:

- Formatting
- Linting
- Testing
- Dependency validation
- Continuous Integration

Automation reduces human error and improves development consistency.

# 3. Development Workflow

The project follows an iterative development process.

Each feature progresses through the following stages:

```text
Planning
    │
    ▼
Design
    │
    ▼
Implementation
    │
    ▼
Testing
    │
    ▼
Code Review
    │
    ▼
Deployment
```

No implementation should begin before the corresponding design has been defined.

## Feature Lifecycle

Every feature should include:

- Business objective
- Technical design
- Unit tests
- Integration tests (when applicable)
- Documentation updates

Features are considered complete only after all supporting artifacts have been updated.

# 4. Git Strategy

The project follows a simplified trunk-based development approach.

The `main` branch always represents the production-ready state of the application.

Development occurs through short-lived feature branches.

Example:

```text
main

feature/project-module

feature/steam-generator

feature/openai-provider

fix/database-timeout
```

Feature branches should remain small and focused.

## Commit Strategy

Commits should represent logical units of work.

Examples:

```text
feat: implement project creation use case

fix: handle OpenAI timeout

refactor: simplify prompt builder

test: add repository integration tests

docs: update backend architecture
```

Commit history should communicate the evolution of the project clearly.

---

# 5. Coding Standards

The backend follows a consistent coding style across all modules.

## Naming

Classes:

- PascalCase

Functions:

- snake_case

Variables:

- snake_case

Constants:

- UPPER_SNAKE_CASE

Modules:

- lowercase

## Function Size

Functions should remain focused on a single responsibility.

As a general guideline:

- Prefer fewer than 30 lines.
- Avoid deeply nested conditionals.
- Return early when possible.

## Documentation

Public modules, classes, and complex functions should include concise documentation.

Documentation should explain **why**, not simply **what**.

Code should remain self-explanatory whenever possible.

## Dependency Direction

The dependency rule defined in `architecture.md` must never be violated.

Presentation depends on Application.

Application depends on Domain.

Infrastructure implements Domain abstractions.

The Domain Layer remains completely independent.

# 6. Dependency Management

Python dependencies are managed using **uv**.

Dependencies should be:

- Explicit
- Versioned
- Reproducible

Unused dependencies should be removed regularly.

The project should avoid introducing large frameworks unless they provide clear long-term value.

Application startup time and dependency footprint should remain small.

# 7. Docker Strategy

Docker is used to guarantee reproducible development and deployment environments.

The backend should run identically on:

- Local development
- Continuous Integration
- Production

Docker images should remain:

- Small
- Deterministic
- Secure

Multi-stage builds should be used whenever practical to minimize image size and reduce attack surface.

The application should never depend on machine-specific configuration.

# 8. Continuous Integration & Continuous Deployment

## Overview

GamePitch AI adopts an automation-first approach for validating every change before it reaches the main branch.

Continuous Integration (CI) ensures that the codebase remains stable, while Continuous Deployment (CD) prepares the application for reliable production releases.

The initial implementation uses **GitHub Actions** as the CI/CD platform.

## Continuous Integration

Every Pull Request should automatically execute the following pipeline:

```text
Checkout Repository
        │
        ▼
Install Dependencies
        │
        ▼
Code Formatting Check
        │
        ▼
Static Analysis
        │
        ▼
Unit Tests
        │
        ▼
Integration Tests
        │
        ▼
Build Docker Image
        │
        ▼
Pipeline Result
```

A Pull Request should not be merged if any required check fails.

## Continuous Deployment

Production deployments are intentionally excluded from the MVP.

However, the architecture is prepared for future automated deployments through GitHub Actions.

Future deployment stages may include:

- Build Docker image
- Push image to container registry
- Deploy to cloud platform
- Execute health checks
- Roll back if deployment fails

Deployment automation should minimize manual intervention.

# 9. Observability

## Overview

Observability provides visibility into the behavior and health of the application in production.

Rather than relying solely on logs, the system combines logs, metrics, and health checks to simplify troubleshooting and operational monitoring.

## Logging

The application uses structured logging.

Each log entry should contain contextual information such as:

- Timestamp
- Request ID
- HTTP method
- Endpoint
- Response status
- Execution time
- AI provider
- Error details (when applicable)

Logs should be machine-readable and suitable for aggregation platforms.

## Metrics

Operational metrics should be collected to monitor application health and business performance.

Examples include:

### Application Metrics

- Request count
- Response latency
- Error rate
- Active requests

### AI Metrics

- Provider response time
- Token usage
- Failed generations
- Average generation duration

### Database Metrics

- Connection pool usage
- Query duration
- Failed queries

Metrics should support proactive performance analysis and capacity planning.

## Health Checks

Health endpoints should expose the operational state of the application.

Examples include:

- Liveness
- Readiness
- Dependency status

These endpoints enable orchestration platforms to detect failures automatically.

# 10. Feature Flags

## Purpose

Feature Flags allow new functionality to be introduced safely without requiring immediate public release.

This enables incremental delivery, experimentation, and rapid rollback.

## Initial Use Cases

Feature Flags may be used to control:

- Experimental generators
- AI provider selection
- Beta features
- New prompt versions
- Internal testing capabilities

## Principles

Feature Flags should be:

- Temporary
- Clearly documented
- Easy to remove
- Environment-aware

Old flags should not remain in the codebase indefinitely.

# 11. Technical Documentation

Documentation is considered part of the product.

Every significant architectural or engineering decision should be documented.

The project documentation consists of:

```text
README.md

docs/
    architecture.md
    backend.md
    engineering.md
    roadmap.md
```

Additional diagrams are stored under:

```text
docs/
    diagrams/
```

Supporting images are stored under:

```text
docs/
    assets/
```

Documentation should evolve alongside the codebase.

## Architecture Decision Records (ADR)

Significant architectural decisions should be captured using Architecture Decision Records.

Each ADR should include:

- Context
- Problem
- Options considered
- Decision
- Positive consequences
- Negative consequences
- Future alternatives

Examples include:

- Why FastAPI was selected
- Why PostgreSQL was chosen
- Why Clean Architecture was adopted
- Why a Modular Monolith was preferred
- Why OpenAI providers are abstracted

ADRs provide historical context and simplify future architectural discussions.

## C4 Model

System documentation follows the C4 Model to describe architecture at multiple levels.

The project maintains diagrams for:

- System Context
- Container
- Component

These diagrams should remain synchronized with the implementation.

## Tech Radar

A lightweight Tech Radar helps communicate the maturity of technologies used within the project.

Technologies are classified into four categories:

### Adopt

Technologies actively recommended.

Examples:

- FastAPI
- PostgreSQL
- Docker
- Pytest
- Pydantic

### Trial

Technologies currently being evaluated.

Examples:

- OpenTelemetry
- Redis
- Background Workers

### Assess

Technologies under investigation.

Examples:

- Vector Databases
- LangGraph
- Temporal

### Hold

Technologies intentionally postponed until business needs justify adoption.

Examples:

- Kubernetes
- Event Streaming Platforms
- Microservices

The Tech Radar should be reviewed periodically as the project evolves.

# 12. Definition of Done

A feature is considered complete only when all of the following criteria have been satisfied.

## Functional Requirements

- Business requirements implemented
- Acceptance criteria satisfied
- API behavior verified

## Code Quality

- Code reviewed
- No known defects
- Architecture respected
- No unnecessary complexity introduced

## Testing

- Unit tests passing
- Integration tests passing (when applicable)
- End-to-end tests updated (when applicable)

## Documentation

- Documentation updated
- API changes documented
- Diagrams updated if necessary

## Operational Readiness

- Logging implemented
- Error handling verified
- Health checks unaffected
- Configuration updated

## Deployment Readiness

- CI pipeline passing
- Docker image builds successfully
- No blocking issues identified

Only after all criteria have been met should a feature be considered complete.

# References

This engineering guide is influenced by established software engineering practices and industry standards, including:

- Clean Architecture
- Domain-Driven Design
- Twelve-Factor App
- C4 Model
- SOLID Principles
- GitHub Flow
- Trunk-Based Development
- FastAPI Documentation
- Docker Documentation
- GitHub Actions Documentation
- OpenTelemetry Documentation
