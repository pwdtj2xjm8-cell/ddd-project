<!--
================================================================================
SYNC IMPACT REPORT
================================================================================
Version change: N/A → 1.0.0 (Initial creation)

Modified principles: N/A (Initial creation)

Added sections:
- Core Principles (5 principles)
  - I. Domain-Driven Design
  - II. Clean Architecture
  - III. Test-First Development (NON-NEGOTIABLE)
  - IV. Containerization & Infrastructure
  - V. Clean Code & Simplicity
- Technical Standards
- Development Workflow
- Governance

Removed sections: N/A

Templates requiring updates:
- .specify/templates/plan-template.md: ✅ Compatible (Constitution Check section exists)
- .specify/templates/spec-template.md: ✅ Compatible (User scenarios align with TDD principle)
- .specify/templates/tasks-template.md: ✅ Compatible (Test-first workflow supported)

Follow-up TODOs: None
================================================================================
-->

# DDD Microservice Constitution

## Core Principles

### I. Domain-Driven Design

The domain model is the heart of the application. All business logic MUST reside within
the domain layer, isolated from infrastructure concerns.

**Non-negotiable rules:**
- Bounded Contexts MUST be clearly defined and documented before implementation
- Aggregates MUST enforce consistency boundaries; only one aggregate per transaction
- Entities MUST have identity; Value Objects MUST be immutable
- Domain Events MUST be used for cross-aggregate communication
- Ubiquitous Language MUST be used consistently in code, tests, and documentation

**Rationale:** DDD ensures the codebase reflects business reality, reducing translation
errors between stakeholders and developers.

### II. Clean Architecture

Dependencies MUST point inward. The domain layer has zero dependencies on infrastructure.

**Non-negotiable rules:**
- Layer structure: Domain → Application → Infrastructure → Presentation
- Domain layer MUST NOT import from infrastructure (SQLAlchemy, HTTP, etc.)
- Use Cases (Application Services) orchestrate domain objects
- Repositories define interfaces in domain; implementations live in infrastructure
- Dependency Injection MUST be used to provide infrastructure to application layer

**Rationale:** Clean Architecture enables testability, flexibility to swap technologies,
and protection of business logic from framework changes.

### III. Test-First Development (NON-NEGOTIABLE)

All production code MUST be written test-first using pytest. No exceptions.

**Non-negotiable rules:**
- Red-Green-Refactor cycle MUST be followed strictly
- Tests MUST be written and FAIL before implementation begins
- Unit tests for domain logic (no mocks for domain objects)
- Integration tests for repository implementations and external services
- Contract tests for API endpoints
- Minimum 80% code coverage for domain and application layers

**Rationale:** TDD produces better design, living documentation, and confidence in
refactoring. It is the primary quality gate for this project.

### IV. Containerization & Infrastructure

All services MUST be containerized with Docker. Infrastructure as Code is mandatory.

**Non-negotiable rules:**
- Every service MUST have a Dockerfile with multi-stage builds
- docker-compose.yml MUST define the complete local development environment
- Environment configuration via environment variables only (12-factor app)
- Health checks MUST be implemented for all services
- Database migrations MUST be versioned and automated (Alembic)

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
**ORM:** SQLAlchemy 2.0+ with async support where applicable
**Testing:** pytest with pytest-mock for test doubles
**Containerization:** Docker with docker-compose for local development
**Code Quality:** Type hints mandatory; linting with ruff or flake8; formatting with black

**Project Structure:**
```
src/
├── domain/           # Entities, Value Objects, Domain Services, Events
├── application/      # Use Cases, DTOs, Application Services
├── infrastructure/   # Repositories, External Services, Database
└── presentation/     # API endpoints, CLI, Controllers

tests/
├── unit/            # Domain and Application layer tests
├── integration/     # Repository and infrastructure tests
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

**Version**: 1.0.0 | **Ratified**: 2026-01-22 | **Last Amended**: 2026-01-22
