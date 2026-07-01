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

# 5. C4 Model

The C4 Model provides multiple levels of abstraction for describing the software architecture.

Each level focuses on a different audience and answers a different architectural question.

For the MVP, GamePitch AI adopts the first three C4 levels.

## Level 1 – System Context

### Purpose

The System Context Diagram describes how GamePitch AI interacts with external actors and third-party systems.

At this level, the application is treated as a single system.

### Primary Actors

- Indie Game Developer
- Small Game Studio
- Future Administrator

### External Systems

- OpenAI API
- OpenRouter
- PostgreSQL
- Authentication Provider (Future)
- Payment Provider (Future)

### Context Diagram

```text
                    +----------------------+
                    |  Indie Game Developer |
                    +-----------+----------+
                                |
                                |
                                v
                    +----------------------+
                    |     GamePitch AI     |
                    |                      |
                    | AI Marketing System  |
                    +----------+-----------+
                               |
                 -------------------------------
                 |                             |
                 |                             |
                 v                             v
        +----------------+          +------------------+
        | OpenAI API     |          | PostgreSQL       |
        +----------------+          +------------------+
```

The user interacts only with GamePitch AI.

All infrastructure concerns remain hidden behind the application's boundaries.

## Level 2 – Container Diagram

### Purpose

The Container Diagram decomposes the application into deployable units.

Although the MVP is deployed as a single application, responsibilities remain clearly separated.

### Container Diagram

```text
                    +----------------------+
                    |      React UI        |
                    +----------+-----------+
                               |
                               |
                               v
                    +----------------------+
                    |     FastAPI API      |
                    +----------+-----------+
                               |
                  ------------------------------
                  |                            |
                  |                            |
                  v                            v
        +--------------------+      +----------------------+
        | PostgreSQL         |      | AI Provider          |
        | Persistence        |      | OpenAI/OpenRouter    |
        +--------------------+      +----------------------+
```

### Container Responsibilities

#### React Frontend

Responsible for:

- User interaction
- Input forms
- Displaying generated content
- API communication

#### FastAPI Backend

Responsible for:

- Business logic
- Prompt orchestration
- AI communication
- Persistence
- Validation
- Logging

#### PostgreSQL

Stores:

- Projects
- Prompt generations
- User data (future)
- Audit information

#### AI Provider

Responsible for generating marketing content.

The provider remains replaceable through the abstraction layer.

## Level 3 – Component Diagram

### Purpose

The Component Diagram illustrates the internal organization of the backend.

This is the primary view for backend developers.

### Component Diagram

```text
                    FastAPI Controllers
                              │
                              ▼
                     Application Services
                              │
          -----------------------------------------
          │                 │                     │
          ▼                 ▼                     ▼
   Prompt Engine     Project Service     Generation Service
          │                 │                     │
          ▼                 ▼                     ▼
      Domain Model     Repository Interfaces      │
          │                                       │
          ▼                                       ▼
     Infrastructure Layer                AI Provider Adapter
          │                                       │
          ▼                                       ▼
     PostgreSQL                         OpenAI/OpenRouter
```

Each component has a single responsibility.

Business logic should never bypass the Application Layer.

Infrastructure components should never communicate directly with one another without passing through business services.

# 6. Core Components

The backend is composed of several logical components that collaborate through explicit interfaces.

Each component has a well-defined responsibility.

## API Layer

### Responsibilities

- HTTP routing
- Request validation
- Response serialization
- Exception mapping
- Authentication (future)

The API Layer should remain thin.

Controllers should never contain business logic.

## Application Layer

The Application Layer orchestrates business workflows.

Responsibilities include:

- Executing use cases
- Coordinating repositories
- Calling the Prompt Engine
- Invoking AI providers
- Managing transactions

Application Services coordinate work but do not contain infrastructure code.

## Prompt Engine

The Prompt Engine is one of the most important components of the platform.

Responsibilities include:

- Selecting prompt templates
- Injecting user variables
- Applying prompt strategies
- Managing prompt versions (future)
- Supporting prompt experimentation

The Prompt Engine is intentionally isolated to allow continuous evolution without affecting the rest of the application.

## Domain Layer

The Domain Layer contains the core business knowledge.

Examples include:

- Project
- Generation
- Prompt Type
- Business Rules
- Validation Policies

This layer must remain completely framework-independent.

## Repository Layer

Repositories abstract data persistence.

Responsibilities include:

- Saving projects
- Retrieving generations
- Querying historical data

Repositories expose contracts rather than implementation details.

## Infrastructure Layer

Infrastructure implements external dependencies.

Examples include:

- PostgreSQL
- Logging
- Configuration
- AI providers
- File storage (future)

Infrastructure may change frequently.

Business logic should not.

## AI Provider Adapter

This component encapsulates communication with external AI services.

Responsibilities include:

- Building provider requests
- Parsing provider responses
- Retry policies
- Timeout handling
- Provider-specific configuration

The Application Layer should never know which provider generated the response.

# 7. Request Flow

The following sequence describes a typical content generation request.

## Step 1

The user submits game information from the frontend.

Input includes:

- Name
- Genre
- Description
- Mechanics
- Target platform

## Step 2

The frontend sends a request to the backend API.

```http
POST /api/v1/generations
```

## Step 3

The controller validates the request.

Invalid requests are rejected before entering the business layer.

## Step 4

The Application Service executes the corresponding use case.

Responsibilities include:

- Validate business rules
- Select generator
- Invoke Prompt Engine

## Step 5

The Prompt Engine builds the final prompt.

Inputs include:

- Prompt template
- User data
- Generator configuration

Output:

Complete prompt ready for inference.

## Step 6

The AI Provider Adapter sends the prompt to the configured provider.

Possible providers include:

- OpenAI
- OpenRouter
- Anthropic (future)

## Step 7

The provider returns generated content.

The adapter normalizes the response into the application's internal format.

## Step 8

The Application Layer stores generation metadata.

Future versions may also persist:

- Prompt version
- Token usage
- Latency
- Cost estimation

## Step 9

The API returns a standardized response.

```json
{
  "content": "...generated text..."
}
```

## Request Sequence

```text
User
 │
 ▼
React
 │
 ▼
FastAPI Controller
 │
 ▼
Application Service
 │
 ▼
Prompt Engine
 │
 ▼
AI Provider Adapter
 │
 ▼
OpenAI
 │
 ▼
Application Service
 │
 ▼
Repository
 │
 ▼
Response
 │
 ▼
React
```

This flow intentionally keeps every component focused on a single responsibility, minimizing coupling and simplifying future evolution.

# 8. Bounded Contexts

Although GamePitch AI is initially implemented as a Modular Monolith, the domain is organized into well-defined bounded contexts. This approach establishes clear ownership boundaries, minimizes coupling, and enables future extraction into independent services if business growth requires it.

Each context encapsulates its own business rules, models, and application services.

## Project Context

The Project Context manages the information required to describe a game.

### Responsibilities

- Create projects
- Update project information
- Validate project data
- Manage project lifecycle

### Main Entities

- Project

## Content Generation Context

The Content Generation Context is responsible for orchestrating AI-powered content generation.

### Responsibilities

- Execute generation requests
- Select the appropriate generator
- Coordinate prompt construction
- Invoke AI providers
- Normalize generated responses

### Main Entities

- Generation
- GenerationType

## Prompt Management Context

Prompt Management centralizes all prompt templates and generation strategies.

Although prompts will initially be stored within the application, the architecture allows future migration to an external Prompt Management Service without impacting business logic.

### Responsibilities

- Prompt selection
- Prompt composition
- Template management
- Prompt versioning (future)
- Prompt experimentation (future)

## User Context (Future)

The User Context will be introduced once authentication becomes part of the platform.

Future responsibilities include:

- User management
- Authentication
- Authorization
- Preferences
- Usage limits

## Billing Context (Future)

Once the application evolves into a SaaS platform, Billing will become an independent bounded context.

Responsibilities may include:

- Subscription management
- Usage tracking
- Credit consumption
- Payment processing
- Invoice generation

# 9. Integration Strategy

GamePitch AI communicates with external systems exclusively through abstraction layers.

This strategy minimizes vendor lock-in and improves maintainability.

## AI Provider Integration

External AI providers are accessed through a common interface.

```text
Application Layer
        │
        ▼
AIProvider Interface
        │
  ┌─────┴───────────────┐
  ▼                     ▼
OpenAI Adapter   OpenRouter Adapter
```

Future providers should only require implementing the interface.

Business logic remains unchanged.

## Database Integration

Persistence is abstracted through repository interfaces.

```text
Application Layer
        │
        ▼
Repository Interface
        │
        ▼
PostgreSQL Repository
```

Future migrations to another relational database should require minimal application changes.

## External Services

Future integrations may include:

- Authentication providers
- Payment gateways
- Email providers
- Analytics platforms
- Object storage services

Each integration should follow the Adapter pattern.

# 10. Scalability Strategy

Scalability should be driven by business needs rather than anticipated complexity.

The MVP intentionally favors simplicity while preserving a clear migration path.

## Application Scalability

Initial deployment consists of a single backend instance.

Future improvements may include:

- Horizontal API scaling
- Load balancing
- Stateless application instances
- Distributed caching

## AI Scalability

AI requests represent the primary performance bottleneck.

Future optimizations include:

- Prompt caching
- Request deduplication
- Background processing
- Provider fallback
- Queue-based execution for long-running tasks

## Database Scalability

The initial PostgreSQL instance is expected to support MVP traffic comfortably.

As demand grows, potential improvements include:

- Read replicas
- Connection pooling
- Query optimization
- Index tuning
- Table partitioning

## Architectural Evolution

The intended evolution path is:

```text
Modular Monolith
        │
        ▼
Distributed Monolith
        │
        ▼
Selective Microservices (if justified)
```

Microservices should only be introduced when supported by measurable business requirements.

# 11. Deployment Architecture

## Development Environment

Local development uses Docker Compose to provide a reproducible environment.

Components include:

- Frontend
- Backend
- PostgreSQL

Developers should be able to bootstrap the project with minimal setup.

## Production Environment

Production deployments are containerized.

Key characteristics include:

- Immutable containers
- Environment-based configuration
- Health checks
- Structured logging
- Automated deployments
- Zero-downtime deployment strategy (future)

The deployment platform should remain cloud-agnostic.

# 12. Cross-Cutting Concerns

Several concerns affect the entire system regardless of individual modules.

## Logging

Structured logging will be implemented across all application layers.

Logs should contain contextual information such as:

- Request identifier
- Execution time
- Endpoint
- User (when applicable)
- AI provider
- Error details

## Observability

Operational visibility will be achieved through:

- Metrics
- Health checks
- Distributed tracing (future)
- Performance monitoring

Observability is considered a first-class engineering capability.

## Security

Security principles include:

- Environment-based secrets
- Secure HTTP communication
- Input validation
- Output sanitization
- Rate limiting
- Principle of least privilege

Security should be integrated into every architectural layer.

## Configuration

Application configuration is externalized.

Environment variables control:

- Database connections
- AI providers
- Secrets
- Feature flags
- Logging levels

No environment-specific configuration should exist inside the source code.

## Feature Flags

Feature Flags enable controlled feature rollout without requiring deployments.

Potential use cases include:

- Experimental generators
- AI provider selection
- Beta features
- A/B testing
- Incremental releases

# 13. Architectural Constraints

The architecture intentionally operates under the following constraints.

## Product Constraints

- MVP-first approach
- Small development team
- Fast delivery cycles
- Limited operational budget

## Technical Constraints

- Single deployable application
- Relational database
- HTTP-based communication
- Cloud-ready deployment

## Business Constraints

- AI inference cost optimization
- Easy onboarding
- Minimal infrastructure complexity
- Rapid feature iteration

These constraints guide architectural decisions throughout the project lifecycle.

# 14. Future Evolution

GamePitch AI is designed to evolve incrementally without requiring architectural rewrites.

Potential future capabilities include:

- Multiple AI providers
- User authentication
- Team collaboration
- Prompt versioning
- Prompt evaluation
- Analytics dashboard
- Background workers
- Event-driven processing
- Public API
- Mobile applications
- Subscription management

The architecture intentionally favors evolutionary design over large-scale redesigns.

# 15. References

The following architectural principles and resources influenced the system design.

- Clean Architecture — Robert C. Martin
- Domain-Driven Design — Eric Evans
- C4 Model — Simon Brown
- Twelve-Factor App Methodology
- SOLID Principles
- REST Architectural Style
- FastAPI Documentation
- PostgreSQL Documentation
- OpenAI API Documentation
- OpenTelemetry Documentation
- Docker Documentation
- GitHub Actions Documentation
