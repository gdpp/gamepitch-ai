# System Architecture

| Field            | Value               |
| ---------------- | ------------------- |
| **Project**      | GamePitch AI        |
| **Document**     | System Architecture |
| **Version**      | 1.0                 |
| **Status**       | Draft               |
| **Owner**        | Gustavo Perez       |
| **Last Updated** | 2026-06-30          |

# Table of Contents

1. Purpose
2. Quality Attributes
3. High-Level Architecture
4. Architectural Style
5. C4 Model
6. Core Components
7. Request Flow
8. Bounded Contexts
9. Integration Strategy
10. Scalability Strategy
11. Deployment Architecture
12. Cross-Cutting Concerns
13. Architectural Constraints
14. Future Evolution
15. References

# 1. Purpose

## Overview

GamePitch AI is designed as a production-ready Software-as-a-Service (SaaS) platform that leverages Large Language Models (LLMs) to generate marketing content tailored specifically for independent game developers and small studios.

Unlike a proof of concept or tutorial project, the architecture prioritizes long-term maintainability, scalability, and operational excellence from the beginning while intentionally avoiding unnecessary complexity during the MVP phase.

The system follows a modular architecture that allows new AI generators, integrations, and business capabilities to be introduced with minimal impact on existing components.

The primary objective is to establish a solid engineering foundation capable of evolving from a single-feature MVP into a complete AI-powered marketing platform.

## Objectives

The architecture has been designed to satisfy the following technical objectives:

### Maintainability

Business rules must remain independent from frameworks, infrastructure, and third-party providers. This allows individual layers to evolve without affecting the entire system.

### Scalability

The application should support increasing workloads by scaling individual components when necessary while keeping the initial deployment model simple.

### Extensibility

Adding a new content generator (such as Steam Tags, Kickstarter Campaigns, or Publisher Pitches) should require minimal modifications to the existing codebase.

### Provider Independence

The application should not depend directly on a single AI provider.

The architecture must allow switching between providers such as:

- OpenAI
- OpenRouter
- Anthropic
- Local LLMs

without requiring changes to the business logic.

### Testability

Business logic should be fully testable without requiring network access, databases, or external AI services.

This enables reliable unit testing and continuous integration.

### Operational Readiness

The system should include production-ready operational capabilities, including:

- Structured logging
- Health checks
- Metrics
- Graceful shutdown
- Configuration management
- Observability
- Automated deployments

These concerns are treated as first-class architectural requirements rather than optional additions.

## Architectural Drivers

Architectural Drivers represent the key factors influencing every technical decision throughout the project.

### Business Driver

The primary business goal is reducing the time and effort required for indie developers to create professional marketing content.

Every architectural decision should contribute to delivering this value efficiently.

### Product Driver

The MVP intentionally focuses on solving one problem exceptionally well:

Generate high-quality Steam descriptions using AI.

Future capabilities should build upon this foundation without requiring major architectural changes.

### Technical Driver

The system must demonstrate production-grade engineering practices suitable for modern backend development.

This includes:

- Clean Architecture
- Dependency Inversion
- SOLID principles
- Modular design
- API-first development
- Automated testing
- Continuous Integration
- Infrastructure as Code readiness

### Operational Driver

The application should be deployable using modern cloud-native practices while remaining simple enough to run locally using Docker Compose.

Operational simplicity is considered a key design principle.

### Cost Driver

Since AI inference represents the primary operational cost, the architecture must enable future optimizations such as:

- Prompt optimization
- Response caching
- Request quotas
- Token monitoring
- Multiple provider support
- Usage analytics

Cost efficiency should evolve alongside product growth.

# 2. Quality Attributes

The architecture is driven by a set of non-functional requirements that define the overall quality of the system.

These attributes serve as the foundation for all architectural decisions.

## Maintainability

The system should be easy to understand, modify, and extend.

To achieve this, the project adopts:

- Clean Architecture
- Clear separation of concerns
- Small, focused modules
- Dependency inversion
- Explicit interfaces
- Consistent coding standards

Business logic should remain isolated from infrastructure and external frameworks.

## Scalability

The MVP is intentionally designed as a Modular Monolith.

This provides a simple deployment model while preserving clear module boundaries that allow future extraction into independent services if required.

Scalability considerations include:

- Stateless API design
- Horizontal scaling
- Externalized configuration
- AI provider abstraction
- Independent persistence layer

## Reliability

The application should continue operating predictably under normal and abnormal conditions.

Reliability is achieved through:

- Centralized exception handling
- Health check endpoints
- Graceful shutdown
- Retry strategies (where appropriate)
- Input validation
- Fault isolation

## Performance

The platform should deliver AI-generated content with minimal latency while optimizing infrastructure costs.

Performance strategies include:

- Asynchronous request handling
- Efficient prompt construction
- Connection pooling
- Future response caching
- Database indexing
- Lightweight API endpoints

Performance optimizations should never compromise code readability or maintainability.

## Security

Security is incorporated as a core architectural concern rather than an afterthought.

Key principles include:

- Secret management through environment variables
- Input validation
- Output sanitization
- Rate limiting
- Secure API communication
- Principle of least privilege
- Dependency vulnerability monitoring

No sensitive credentials should ever be stored in source control.

## Testability

Every business rule should be independently testable.

The testing strategy favors:

- Unit tests
- Integration tests
- End-to-end tests where appropriate
- Dependency injection
- Mockable interfaces
- Isolated business logic

Testing should provide confidence for continuous deployment.

## Observability

Operational visibility is essential for maintaining production systems.

The platform should provide:

- Structured logs
- Request tracing
- Performance metrics
- Health monitoring
- Error reporting
- AI usage metrics

Observability enables proactive issue detection and simplifies production debugging.

## Simplicity

Although the architecture targets production environments, unnecessary complexity should be avoided.

The project follows the principle of:

> Build for today's requirements while preparing for tomorrow's growth.

This means avoiding premature adoption of distributed systems, message brokers, or microservices until clear business requirements justify their introduction.

The MVP should remain simple, understandable, and easy to evolve.

# 3. High-Level Architecture

## Overview

GamePitch AI follows a layered, API-first architecture designed around the principles of Clean Architecture and Modular Monolith.

The system separates business rules from infrastructure concerns, allowing individual components to evolve independently while maintaining a simple deployment model during the MVP.

At a high level, the platform consists of five major components:

- React Frontend
- FastAPI Backend
- Application Core
- Persistence Layer
- AI Provider Integration

Each component has a single responsibility and communicates through clearly defined interfaces.

## High-Level Component Diagram

```text
                           +----------------------+
                           |      React UI        |
                           |  (Frontend Client)   |
                           +----------+-----------+
                                      |
                                      | HTTPS / REST
                                      |
                           +----------v-----------+
                           |      FastAPI API     |
                           |   Presentation Layer |
                           +----------+-----------+
                                      |
                        -------------------------------
                        |                             |
                        |                             |
             +----------v-----------+      +----------v-----------+
             |   Application Layer  |      |    Health & Config   |
             |  Use Cases / Services |      | Logging / Metrics    |
             +----------+-----------+      +----------------------+
                        |
             +----------v-----------+
             |    Domain Layer      |
             | Business Rules       |
             | Entities             |
             | Interfaces           |
             +----------+-----------+
                        |
          -------------------------------------
          |                                   |
+---------v---------+              +----------v----------+
| Infrastructure    |              | AI Provider Adapter |
| PostgreSQL        |              | OpenAI/OpenRouter   |
| Repositories      |              | Anthropic (Future)  |
+-------------------+              +---------------------+
```

## Architectural Responsibilities

### Frontend

Responsible for:

- User interface
- Input validation
- User interactions
- Displaying generated content
- Communicating with the backend API

The frontend contains no business logic related to AI generation.

### Backend API

Acts as the entry point to the system.

Responsibilities include:

- Request validation
- Authentication (future)
- Routing
- API versioning
- Error handling
- Response formatting

The API should remain thin and delegate business operations to the Application Layer.

### Application Layer

Coordinates business workflows.

Responsibilities include:

- Executing use cases
- Orchestrating repositories
- Calling AI providers
- Managing transactions
- Applying business policies

This layer contains application-specific logic but no infrastructure concerns.

### Domain Layer

The Domain Layer represents the core of the application.

It contains:

- Entities
- Value Objects
- Business Rules
- Repository Contracts
- Service Interfaces

The Domain Layer must not depend on any framework, database, or external service.

### Infrastructure Layer

Responsible for technical implementation details.

Examples include:

- PostgreSQL repositories
- AI provider implementations
- External APIs
- Logging providers
- Configuration management
- File storage (future)

Infrastructure depends on the Domain, but the Domain never depends on Infrastructure.

# 4. Architectural Style

## Clean Architecture

GamePitch AI adopts Clean Architecture as its primary architectural style.

This approach ensures that business rules remain independent from external technologies, making the system easier to maintain, test, and evolve.

Dependencies always point inward.

```text
Frameworks
      ↓

Infrastructure
      ↓

Application
      ↓

Domain
```

The Domain Layer represents the center of the system and contains the business knowledge that should remain stable over time.

## Why Clean Architecture?

Several architectural styles were evaluated before selecting Clean Architecture.

The primary reasons include:

- Clear separation of concerns
- Excellent testability
- Framework independence
- Long-term maintainability
- Easy replacement of infrastructure
- Improved code organization

For an AI-driven SaaS expected to evolve continuously, maintaining isolated business rules significantly reduces future development costs.

## API-First Design

The backend is designed as an API-first system.

Every business capability is exposed through HTTP APIs before considering frontend implementation.

Benefits include:

- Frontend independence
- Easier testing
- External integrations
- Mobile application support
- Future public API possibilities

The API represents the contract between clients and the business layer.

## Modular Monolith

Although the system is expected to grow over time, the MVP intentionally adopts a Modular Monolith architecture.

This approach offers several advantages:

- Simpler deployments
- Lower operational costs
- Easier debugging
- Strong module boundaries
- No distributed system complexity

Each business capability remains isolated inside its own module, allowing future extraction into independent services if business growth requires it.

Current modules include:

- Projects
- Content Generation
- Prompt Management

Future modules may include:

- Authentication
- Billing
- Analytics
- Notifications
- Subscription Management

## Dependency Rule

The architecture follows the Dependency Inversion Principle.

High-level modules should never depend directly on low-level implementations.

Instead, dependencies are inverted through interfaces.

```text
Application Service
        │
        ▼

Prompt Repository (Interface)

        ▲
        │

PostgreSQL Repository
```

The business layer depends only on abstractions.

This enables infrastructure components to be replaced without affecting business logic.

## AI Provider Abstraction

The application should never communicate directly with a specific AI provider.

Instead, all providers implement a common contract.

```text
Application Service
        │
        ▼

AIProvider Interface

      ▲      ▲      ▲
      │      │      │

OpenAI   OpenRouter   Anthropic
```

Benefits include:

- Provider independence
- Easier testing
- Vendor flexibility
- Reduced lock-in
- Support for local LLMs

Future providers can be integrated without modifying existing business logic.

## Configuration Strategy

Application configuration should be externalized.

The system follows the Twelve-Factor App principle of storing configuration outside the codebase.

Configuration includes:

- Database connection
- AI provider selection
- API keys
- Feature flags
- Logging level
- Environment settings

No environment-specific values should be hardcoded.

## Error Handling Strategy

Errors should propagate through well-defined boundaries.

Unexpected exceptions should never reach the client directly.

The architecture defines three categories of errors:

### Validation Errors

Incorrect client input.

Example:

- Missing required fields
- Invalid request format

### Business Errors

Domain-specific failures.

Example:

- Unsupported content type
- Invalid game configuration

### Infrastructure Errors

Failures caused by external systems.

Examples:

- Database unavailable
- AI provider timeout
- Network failures

Each category should produce standardized API responses and structured logs for troubleshooting.
