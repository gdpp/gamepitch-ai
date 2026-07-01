# Backend Design

| Field            | Value          |
| ---------------- | -------------- |
| **Project**      | GamePitch AI   |
| **Document**     | Backend Design |
| **Version**      | 1.0            |
| **Status**       | Draft          |
| **Owner**        | Gustavo Perez  |
| **Last Updated** | 2026-07-01     |

# Table of Contents

1. Overview
2. Design Principles
3. Technology Stack
4. Backend Structure
5. Layer Responsibilities
6. Dependency Flow
7. Module Organization
8. API Design
9. Data Model
10. Prompt Engine
11. AI Provider Abstraction
12. Error Handling
13. Concurrency Strategy
14. Security
15. References

# 1. Overview

## Purpose

The backend is responsible for orchestrating all business operations of GamePitch AI.

It acts as the central component of the platform by coordinating user requests, executing business rules, communicating with AI providers, persisting application data, and exposing a stable REST API to external clients.

The backend is intentionally designed to remain independent from frontend technologies, allowing additional clients such as mobile applications, CLI tools, or third-party integrations to be introduced in the future.

## Objectives

The backend architecture pursues the following objectives:

- Keep business rules independent from frameworks.
- Support multiple AI providers through abstraction.
- Maintain high testability.
- Enable incremental feature development.
- Remain production-ready from the first release.
- Minimize coupling between modules.
- Allow future migration toward distributed services if required.

# 2. Design Principles

The backend follows a set of engineering principles that guide implementation decisions.

## Clean Architecture

Business rules remain at the center of the system.

Frameworks, databases, external APIs, and infrastructure are implementation details.

## Separation of Concerns

Each layer owns a single responsibility.

Business logic should never appear inside controllers or repositories.

## Dependency Inversion

High-level modules depend only on abstractions.

Infrastructure components implement interfaces defined by the business layer.

## Explicit Boundaries

Each module exposes only its public contracts.

Internal implementation details should remain hidden.

## Simplicity First

The MVP intentionally avoids unnecessary complexity.

The system should evolve only when justified by business requirements.

# 3. Technology Stack

| Layer                 | Technology          |
| --------------------- | ------------------- |
| Language              | Python 3.13+        |
| Framework             | FastAPI             |
| Validation            | Pydantic v2         |
| ORM                   | SQLAlchemy 2.x      |
| Database Migrations   | Alembic             |
| Database              | PostgreSQL          |
| AI Provider           | OpenAI / OpenRouter |
| Dependency Management | uv                  |
| Containerization      | Docker              |
| Testing               | Pytest              |
| HTTP Client           | HTTPX               |
| Logging               | Structlog           |
| Configuration         | Pydantic Settings   |

The selected technologies prioritize long-term maintainability, performance, and strong typing.

# 4. Backend Structure

The backend follows Clean Architecture.

```text
backend/
│
├── src/
│
│   ├── domain/
│   │
│   ├── application/
│   │
│   ├── infrastructure/
│   │
│   ├── presentation/
│   │
│   ├── shared/
│   │
│   └── main.py
│
├── tests/
│
├── migrations/
│
├── scripts/
│
└── pyproject.toml
```

Each directory represents a logical architectural layer rather than a technical grouping.

# 5. Layer Responsibilities

## Domain

The Domain Layer contains the business model.

Responsibilities include:

- Entities
- Value Objects
- Domain Services
- Repository Interfaces
- Business Rules
- Domain Exceptions

The Domain Layer must never depend on FastAPI, SQLAlchemy, PostgreSQL, or external providers.

## Application

The Application Layer coordinates use cases.

Responsibilities include:

- Execute business workflows
- Coordinate repositories
- Manage transactions
- Invoke AI providers
- Build responses

Application Services orchestrate operations but do not contain infrastructure code.

## Infrastructure

Infrastructure contains technical implementations.

Examples include:

- PostgreSQL repositories
- SQLAlchemy models
- AI provider adapters
- Logging
- Configuration
- External services

Infrastructure may change frequently without affecting business rules.

## Presentation

The Presentation Layer exposes the application through HTTP.

Responsibilities include:

- API routes
- Request validation
- Response serialization
- Exception mapping
- API documentation

Controllers should remain thin and delegate business operations to the Application Layer.

## Shared

The Shared module contains reusable technical components.

Examples include:

- Configuration
- Common utilities
- Constants
- Logging
- Middleware
- Feature flags

Only truly cross-cutting components belong here.

# 6. Dependency Flow

Dependencies always point inward.

```text
Presentation
      │
      ▼
Application
      │
      ▼
Domain
      ▲
      │
Infrastructure
```

This dependency rule guarantees that business logic remains isolated from infrastructure concerns.

# 7. Module Organization

Business capabilities are grouped into independent modules rather than technical folders.

The initial MVP includes the following modules.

## Projects

Responsible for managing game information.

Future responsibilities include:

- CRUD operations
- Validation
- Search
- Ownership

## Generations

Responsible for AI-powered content generation.

Responsibilities include:

- Generator selection
- Prompt orchestration
- Provider invocation
- Response normalization

## Prompts

Responsible for prompt construction.

Responsibilities include:

- Template selection
- Variable injection
- Prompt versioning (future)
- Prompt experimentation

## Health

Provides operational endpoints.

Responsibilities include:

- Health checks
- Readiness checks
- Liveness checks

These endpoints support production monitoring and orchestration platforms.

# 8. API Design

## API Philosophy

The backend exposes a RESTful API designed around business capabilities rather than database entities.

Endpoints represent actions that users perform instead of simple CRUD operations.

The API follows these principles:

- Resource-oriented design
- Predictable URL structure
- Versioned endpoints
- Stateless communication
- JSON request and response bodies
- Consistent error responses

The initial version is exposed under:

```text
/api/v1
```

Future breaking changes will be introduced through new API versions without affecting existing clients.

## Endpoint Structure

```text
/api/v1
    ├── /projects
    ├── /generations
    ├── /health
    └── /system
```

Each module owns its own routes and business logic.

## Initial Endpoints

### Projects

| Method | Endpoint         | Description                |
| ------ | ---------------- | -------------------------- |
| POST   | `/projects`      | Create a new game project  |
| GET    | `/projects/{id}` | Retrieve a project         |
| PUT    | `/projects/{id}` | Update project information |
| DELETE | `/projects/{id}` | Delete a project           |

### Generations

| Method | Endpoint                       | Description                    |
| ------ | ------------------------------ | ------------------------------ |
| POST   | `/generations`                 | Generate AI marketing content  |
| GET    | `/generations/{id}`            | Retrieve a previous generation |
| POST   | `/generations/{id}/regenerate` | Generate a new variation       |

### Health

| Method | Endpoint        | Description        |
| ------ | --------------- | ------------------ |
| GET    | `/health`       | Basic health check |
| GET    | `/health/live`  | Liveness probe     |
| GET    | `/health/ready` | Readiness probe    |

## API Response Format

Successful responses follow a predictable structure.

```json
{
  "data": {
    ...
  }
}
```

Errors follow a standardized format.

```json
{
  "error": {
    "code": "GENERATION_FAILED",
    "message": "Unable to generate content."
  }
}
```

A consistent contract simplifies frontend integration and improves developer experience.

# 9. Data Model

The MVP introduces a small but extensible relational model.

The design prioritizes normalization, clarity, and future scalability.

## Project

Represents a video game created by the user.

### Attributes

- id
- name
- genre
- description
- mechanics
- target_platform
- created_at
- updated_at

## Generation

Represents AI-generated marketing content.

### Attributes

- id
- project_id
- generation_type
- content
- provider
- prompt_version
- created_at

## Future Entities

The following entities will be introduced in later phases.

- User
- Organization
- Subscription
- PromptTemplate
- PromptVersion
- UsageRecord
- AuditLog

The MVP intentionally avoids unnecessary complexity while preserving a clear migration path.

## Entity Relationships

```text
Project
   │
   │ 1
   │
   ▼
Generation
```

One project can produce multiple AI generations.

# 10. Prompt Engine

## Overview

The Prompt Engine represents the core business capability of GamePitch AI.

Rather than embedding prompts directly inside controllers or services, prompt construction is centralized into a dedicated component.

This design improves consistency, maintainability, and experimentation.

## Responsibilities

The Prompt Engine is responsible for:

- Selecting prompt templates
- Injecting user variables
- Applying generation strategies
- Supporting tone variations
- Managing prompt versions
- Preparing provider-ready prompts

The Prompt Engine does **not** communicate directly with external AI providers.

Its only responsibility is prompt construction.

## Prompt Lifecycle

```text
Template
      │
      ▼
Variable Injection
      │
      ▼
Prompt Builder
      │
      ▼
Final Prompt
```

Separating prompt generation from provider communication allows prompts to evolve independently.

## Prompt Templates

Prompt templates are treated as versioned business assets.

Examples include:

- Steam Description
- Social Media Post
- Trailer Script
- Publisher Pitch

Future versions may support:

- Localization
- Prompt experiments
- Dynamic prompt composition

# 11. AI Provider Abstraction

The backend should never depend directly on a specific LLM provider.

Instead, all providers implement a common contract.

```text
Application Service
        │
        ▼
AIProvider Interface
        │
  ┌─────┼───────────────┐
  ▼     ▼               ▼
OpenAI OpenRouter Anthropic
```

This approach provides:

- Vendor independence
- Easier testing
- Future extensibility
- Reduced migration costs

## Provider Responsibilities

Every provider implementation must support:

- Prompt submission
- Response normalization
- Timeout handling
- Error translation
- Usage reporting

The Application Layer interacts only with the interface.

# 12. Error Handling

Errors are categorized according to their origin.

## Validation Errors

Generated when client input is invalid.

Examples include:

- Missing fields
- Invalid payloads
- Unsupported values

HTTP Status:

```text
400 Bad Request
```

## Business Errors

Generated when business rules are violated.

Examples include:

- Unsupported generation type
- Invalid project state

HTTP Status:

```text
422 Unprocessable Entity
```

## Infrastructure Errors

Generated by external dependencies.

Examples include:

- Database unavailable
- AI provider timeout
- Network failures

HTTP Status:

```text
500 Internal Server Error
```

## Unexpected Errors

Unhandled exceptions should never reach the client.

Instead:

- Log the exception
- Return a standardized response
- Preserve internal details

This minimizes information leakage while simplifying debugging.

# 13. Concurrency Strategy

## Overview

GamePitch AI is designed to handle multiple concurrent requests efficiently while maintaining a simple execution model during the MVP.

The backend leverages Python's asynchronous programming model through FastAPI and the ASGI specification.

This approach enables efficient handling of I/O-bound operations such as database communication and AI provider requests without introducing unnecessary architectural complexity.

## Asynchronous Processing

The following operations should be implemented asynchronously whenever possible:

- HTTP request handling
- AI provider communication
- Database operations
- External API calls
- Health check dependencies

CPU-intensive work should remain minimal within the API process.

If future business requirements introduce expensive processing workloads, these operations should be delegated to background workers.

## Current Execution Model

```text
HTTP Request
      │
      ▼
FastAPI Endpoint (async)
      │
      ▼
Application Service
      │
      ▼
Repository / AI Provider
      │
      ▼
HTTP Response
```

The MVP intentionally avoids queues and distributed workers to reduce operational complexity.

## Future Evolution

As traffic increases, asynchronous execution may be complemented with:

- Background tasks
- Message queues
- Worker processes
- Scheduled jobs
- Distributed task execution

These additions should only be introduced when justified by measurable business requirements.

# 14. Configuration Strategy

## Overview

Application configuration follows the Twelve-Factor App methodology.

Configuration should never be hardcoded.

Instead, all environment-specific values are provided through environment variables.

## Configuration Categories

The backend configuration includes:

### Application

- Environment
- Debug mode
- API version

### Database

- Host
- Port
- Username
- Password
- Database name

### AI Providers

- Provider selection
- API keys
- Default model
- Request timeout

### Logging

- Log level
- Structured output
- Correlation identifiers

### Feature Flags

- Experimental generators
- Provider selection
- Beta capabilities

## Configuration Loading

Configuration should be loaded once during application startup.

The rest of the application depends on strongly typed configuration objects rather than raw environment variables.

This approach improves reliability and testability.

# 15. Security

## Security Principles

Security is considered a foundational aspect of the backend architecture.

The MVP adopts secure defaults while remaining simple enough for rapid iteration.

## Secrets Management

Sensitive information must never be committed to source control.

Examples include:

- API keys
- Database credentials
- JWT secrets
- Encryption keys

Secrets should be supplied through environment variables or a dedicated secret management solution in production.

## Input Validation

Every incoming request should be validated before entering the business layer.

Validation includes:

- Required fields
- Data types
- Length constraints
- Allowed values

Pydantic models serve as the first validation boundary.

## Rate Limiting

To protect AI resources and control operational costs, rate limiting should be introduced before public release.

Future implementations may enforce:

- Requests per minute
- Daily quotas
- User-based limits
- IP-based throttling

## Authentication

Authentication is intentionally excluded from the MVP.

The architecture, however, reserves clear extension points for introducing:

- JWT authentication
- OAuth providers
- API keys
- Session management

# 16. Logging

## Overview

Structured logging is implemented across all backend layers.

Logs should be machine-readable, searchable, and consistent.

## Logging Strategy

Each request should include contextual information such as:

- Request ID
- Timestamp
- Endpoint
- HTTP method
- Execution time
- Status code
- AI provider
- Exception details (if applicable)

## Log Levels

The application follows standard log levels.

| Level    | Purpose                   |
| -------- | ------------------------- |
| DEBUG    | Development diagnostics   |
| INFO     | Normal application events |
| WARNING  | Recoverable issues        |
| ERROR    | Failed operations         |
| CRITICAL | System failures           |

Logs should avoid exposing sensitive information.

# 17. Health Checks

The backend exposes dedicated endpoints for infrastructure monitoring.

## Health

Verifies that the application process is running.

```text
GET /api/v1/health
```

## Liveness

Used by orchestration platforms to determine whether the application should be restarted.

```text
GET /api/v1/health/live
```

## Readiness

Determines whether the application is capable of receiving traffic.

Checks may include:

- Database connectivity
- AI provider availability (optional)
- Configuration loading

```text
GET /api/v1/health/ready
```

# 18. Graceful Shutdown

The backend should terminate safely when receiving operating system signals.

Shutdown procedures include:

- Finish active requests
- Close database connections
- Flush pending logs
- Release external resources

Graceful shutdown prevents request loss and database inconsistencies during deployments.

# 19. Testing Strategy

Testing is treated as an integral part of the backend architecture rather than an afterthought.

## Unit Tests

Focus on isolated business logic.

Characteristics:

- Fast execution
- No external dependencies
- Mock infrastructure
- High coverage

Target:

- Domain Layer
- Application Layer

## Integration Tests

Validate interactions between multiple components.

Examples include:

- Database repositories
- AI provider adapters
- Configuration loading

Integration tests execute against real infrastructure whenever practical.

## End-to-End Tests

Validate complete user workflows.

Examples:

- Create project
- Generate Steam description
- Persist generation
- Retrieve previous results

End-to-end tests provide confidence that the system behaves correctly as a whole.

## Testing Pyramid

```text
           E2E
            ▲
      Integration
            ▲
        Unit Tests
```

The majority of tests should remain at the unit level.

# 20. Performance Considerations

Performance optimization should be guided by measurement rather than assumption.

Initial optimization strategies include:

- Async request handling
- Efficient SQL queries
- Connection pooling
- HTTP client reuse
- Prompt optimization

Future optimizations may include:

- Redis caching
- Response caching
- Background processing
- Read replicas
- Queue-based execution

Premature optimization should be avoided.

# 21. References

The backend design is influenced by the following engineering practices and resources:

- FastAPI Documentation
- SQLAlchemy Documentation
- Pydantic Documentation
- PostgreSQL Documentation
- Alembic Documentation
- HTTPX Documentation
- Structlog Documentation
- Twelve-Factor App
- Clean Architecture
- Domain-Driven Design
- SOLID Principles
