<!--
================================================================================
SYNC IMPACT REPORT
================================================================================
Version change: 1.0.0 → 1.1.0 (MINOR - materially expanded DDD principle with
Event Sourcing, added domain-specific constraints)

Modified principles:
- I. Domain-Driven Design → I. Domain-Driven Design & Event Sourcing (expanded)

Added sections:
- Event Sourcing Pattern rules under Principle I
- Produktionscharge as single Aggregate Root constraint
- Command/Event/Policy flow specification

Removed sections: None

Technical Standards changes:
- Database: SQLAlchemy → SQLite (replaced)
- Added Event Store requirement

Templates requiring updates:
- .specify/templates/plan-template.md: ✅ Compatible (Constitution Check section exists)
- .specify/templates/spec-template.md: ✅ Compatible (User scenarios align with TDD principle)
- .specify/templates/tasks-template.md: ✅ Compatible (Test-first workflow supported)

Follow-up TODOs: None
================================================================================
-->

# DDD Microservice Constitution

## Core Principles

### I. Domain-Driven Design & Event Sourcing

The domain model is the heart of the application. All business logic MUST reside within
the domain layer, isolated from infrastructure concerns. The system follows Event Sourcing
as the primary persistence pattern.

**Non-negotiable rules:**

**DDD Building Blocks:**
- **Produktionscharge** (Production Batch) is the ONLY Aggregate Root in this domain
- Aggregates MUST enforce consistency boundaries; only one aggregate per transaction
- Invarianten (business rules) MUST be protected and enforced within the Aggregate
- Entities MUST have identity; Value Objects MUST be immutable
- Value Objects MUST be used where semantically appropriate (e.g., quantities, dates, identifiers)
- Ubiquitous Language MUST be used consistently in code, tests, and documentation

**Event Sourcing & CQRS Pattern:**
- Commands MUST produce Events (never modify state directly)
- Events are the single source of truth; state is derived by replaying events
- Policies MUST react to Events and MAY issue new Commands
- Read Models (Projections) MUST be built from Events for query optimization
- External Systems MUST be integrated via Anti-Corruption Layer

**Command/Event Flow:**
```
Command → Aggregate → Event(s) → Event Store
                         ↓
              Policy → new Command (optional)
                         ↓
              Read Model Update (Projection)
                         ↓
              External System Notification (optional)
```

**Rationale:** DDD ensures the codebase reflects business reality, reducing translation
errors between stakeholders and developers. Event Sourcing provides a complete audit trail,
enables temporal queries, and allows rebuilding state from events.

### II. Clean Architecture

Dependencies MUST point inward. The domain layer has zero dependencies on infrastructure.

**Non-negotiable rules:**
- Layer structure: Domain → Application → Infrastructure → Presentation
- Domain layer MUST NOT import from infrastructure (SQLite, HTTP, etc.)
- Use Cases (Application Services) orchestrate domain objects and command handling
- Repositories define interfaces in domain; implementations live in infrastructure
- Event Store interface defined in domain; SQLite implementation in infrastructure
- Dependency Injection MUST be used to provide infrastructure to application layer

**Rationale:** Clean Architecture enables testability, flexibility to swap technologies,
and protection of business logic from framework changes.

### III. Test-First Development (NON-NEGOTIABLE)

All production code MUST be written test-first using pytest. No exceptions.

**Non-negotiable rules:**
- Red-Green-Refactor cycle MUST be followed strictly
- Tests MUST be written and FAIL before implementation begins
- Unit tests for domain logic (Aggregates, Value Objects, Policies - no mocks for domain objects)
- Integration tests for Event Store, repository implementations, and external services
- Contract tests for API endpoints
- Minimum 80% code coverage for domain and application layers
- Event-based tests: verify correct events are produced for given commands

**Rationale:** TDD produces better design, living documentation, and confidence in
refactoring. It is the primary quality gate for this project.

### IV. Containerization & Infrastructure

All services MUST be containerized with Docker. Infrastructure as Code is mandatory.

**Non-negotiable rules:**
- Every service MUST have a Dockerfile with multi-stage builds
- docker-compose.yml MUST define the complete local development environment
- Environment configuration via environment variables only (12-factor app)
- Health checks MUST be implemented for all services
- Database migrations MUST be versioned and automated

**Rationale:** Containers ensure reproducible environments across development, testing,
and production, eliminating "works on my machine" issues.

### V. Clean Code & Simplicity

Code MUST be readable, maintainable, and no more complex than necessary.

**Non-negotiable rules:**
- YAGNI: Do not implement features until they are needed
- DRY: Extract duplication only after it appears three times
- Single Responsibility: Each module, class, and function does one thing well
- Explicit over implicit: No magic; dependencies and behavior must be obvious
- Type hints MUST be used for all function signatures and class attributes
- Docstrings required for public APIs; inline comments only for non-obvious logic

**Rationale:** Simple code is easier to understand, test, modify, and debug. Complexity
must always be justified against the current requirements.

## Technical Standards

**Language/Runtime:** Python 3.11+
**Database:** SQLite (file-based, embedded)
**Event Store:** SQLite-backed event store implementation
**Testing:** pytest with pytest-mock for test doubles
**Containerization:** Docker with docker-compose for local development
**Code Quality:** Type hints mandatory; linting with ruff or flake8; formatting with black

**Project Structure:**
```
src/
├── domain/           # Aggregates, Value Objects, Domain Events, Commands, Policies
│   ├── aggregates/   # Produktionscharge (single aggregate root)
│   ├── events/       # Domain Events
│   ├── commands/     # Command definitions
│   ├── policies/     # Event handlers that may produce commands
│   └── value_objects/# Immutable value objects
├── application/      # Use Cases, Command Handlers, Query Handlers
├── infrastructure/   # Event Store (SQLite), Repositories, External Systems
│   ├── persistence/  # SQLite event store implementation
│   ├── projections/  # Read model builders
│   └── external/     # Anti-corruption layer for external systems
└── presentation/     # API endpoints, CLI, Controllers

tests/
├── unit/            # Domain and Application layer tests
├── integration/     # Event store and infrastructure tests
└── contract/        # API contract tests
```

## Development Workflow

**Branch Strategy:**
- Feature branches from `main`: `feature/[ticket]-description`
- All changes via Pull Request with required review

**Commit Standards:**
- Conventional commits: `feat:`, `fix:`, `refactor:`, `test:`, `docs:`
- Each commit MUST leave tests passing

**Code Review Requirements:**
- All PRs MUST pass CI (tests, linting, type checking)
- Constitution compliance MUST be verified
- At least one approval required before merge

**Definition of Done:**
- [ ] Tests written first and passing
- [ ] Code follows Clean Architecture layers
- [ ] Events properly defined and stored
- [ ] Invariants protected in Aggregate
- [ ] Type hints complete
- [ ] Documentation updated if public API changed
- [ ] Docker build succeeds
- [ ] No linting errors

## Governance

This Constitution is the supreme authority for development practices in this project.
All code, reviews, and architectural decisions MUST comply with these principles.

**Amendment Process:**
1. Propose changes via Pull Request to this document
2. Document rationale for change
3. Obtain team consensus
4. Update version number according to semantic versioning
5. Update dependent templates if principles change

**Versioning Policy:**
- MAJOR: Principle removal or backward-incompatible governance change
- MINOR: New principle added or existing principle materially expanded
- PATCH: Clarifications, wording improvements, typo fixes

**Compliance Review:**
- Every PR MUST include Constitution Check verification
- Quarterly review of Constitution relevance and adoption
- Violations MUST be documented and resolved before merge

**Version**: 1.1.0 | **Ratified**: 2026-01-22 | **Last Amended**: 2026-01-22