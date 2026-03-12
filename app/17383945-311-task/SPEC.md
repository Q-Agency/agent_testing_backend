# SPEC — 311: Markdown Note App Feature
**Type:** [Pending]
**Status:** DRAFT
**Author:** AI-assisted
**Date:** 2026-02-23

## Overview
[Pending — awaiting task scope clarification]

## User Stories
[Pending]

## Acceptance Criteria
[Pending]

## Scope
### In Scope
[Pending]

### Out of Scope
[Pending]

## Business Rules
[Pending]

## Error Handling
**Format:** RFC 7807 Problem Details JSON (`type`, `title`, `status`, `detail`)
**Strategy:** Custom exception classes. Backend exceptions inherit from base `AppError` class.
- User-facing errors use RFC 7807 format
- No logging (POC project)

## Data Model
[Pending]

## Dependencies
**Backend Stack:**
- Language: Python 3.12
- Framework: FastAPI 0.115.x
- Database: SQLite 3.x
- Testing: pytest 8.x
- Linting: Ruff 0.9.x
- Type Checking: mypy 1.14.x

**Frontend Stack:**
- Language: TypeScript 5.7
- Framework: React 19
- Build Tool: Vite 6.x
- Testing: Vitest 3.x
- Styling: TailwindCSS 4.x

**API Standards:**
- Style: REST
- Versioning: `/api/v1/`
- Error Format: RFC 7807 Problem Details
- Content-Type: `application/json`

**Architectural Patterns:**
- Repository pattern for data access
- Pydantic schemas for request/response validation
- FastAPI dependency injection

**Testing Requirements:**
- Minimum coverage: 80%
- Unit tests required
- Integration tests required

## Open Questions
- [ ] What is the primary user journey for this task?
- [ ] Are there specific acceptance criteria or edge cases?
- [ ] What is the priority and timeline?

## Decisions Made
| Question | Decision | Source |
|----------|----------|--------|
| Error handling format | RFC 7807 Problem Details | CONSTITUTION.md |
| Backend framework | FastAPI 0.115.x | CONSTITUTION.md |
| Frontend framework | React 19 + TypeScript | CONSTITUTION.md |
| Database | SQLite 3.x | CONSTITUTION.md |
| API style | REST with `/api/v1/` versioning | CONSTITUTION.md |
