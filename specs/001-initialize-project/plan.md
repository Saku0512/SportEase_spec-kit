# Implementation Plan: SportEase Initial Project Setup

**Branch**: `001-initialize-project` | **Date**: 2025-09-07 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/001-initialize-project/spec.md`

## Execution Flow (/plan command scope)
```
1. Load feature spec from Input path
   → If not found: ERROR "No feature spec at {path}"
2. Fill Technical Context (scan for NEEDS CLARIFICATION)
   → Detect Project Type from context (web=frontend+backend, mobile=app+api)
   → Set Structure Decision based on project type
3. Evaluate Constitution Check section below
   → If violations exist: Document in Complexity Tracking
   → If no justification possible: ERROR "Simplify approach first"
   → Update Progress Tracking: Initial Constitution Check
4. Execute Phase 0 → research.md
   → If NEEDS CLARIFICATION remain: ERROR "Resolve unknowns"
5. Execute Phase 1 → contracts, data-model.md, quickstart.md, agent-specific template file (e.g., `CLAUDE.md` for Claude Code, `.github/copilot-instructions.md` for GitHub Copilot, or `GEMINI.md` for Gemini CLI).
6. Re-evaluate Constitution Check section
   → If new violations: Refactor design, return to Phase 1
   → Update Progress Tracking: Post-Design Constitution Check
7. Plan Phase 2 → Describe task generation approach (DO NOT create tasks.md)
8. STOP - Ready for /tasks command
```

## Summary
仙台高専向けのマイクロサービスベースのスポーツイベント管理システム「SportEase」を構築する。管理者はイベントやユーザーを管理し、学生はQRコードでイベントに参加する。技術アプローチとして、フロントエンドにReact、バックエンドにGoを用いたマイクロサービスアーキテクチャを採用し、DockerとGCP上で実行する。

## Technical Context
**Language/Version**: **Go 1.21+**, **TypeScript 5.x**
**Primary Dependencies**: **Go**: net/http, gorilla/mux, pq (for PostgreSQL). **React**: Vite, MUI, react-router-dom.
**Storage**: **PostgreSQL 15+**
**Testing**: **Go**: standard testing package. **React**: Vitest, React Testing Library.
**Target Platform**: Web browsers on **Linux** (for backend services on GCP).
**Project Type**: **web**
**Performance Goals**: p99 latency < 200ms for critical API requests.
**Constraints**: Google認証は `@sendai-nct.jp` ドメインのみ許可。
**Scale/Scope**: ~700 total users, ~200 concurrent users during peak event times.

## Constitution Check
*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

**Simplicity**:
- Projects: **6+** (frontend, api-gateway, auth-service, user-service, event-service, etc.) -> **VIOLATION** (see Complexity Tracking)
- Using framework directly? **Yes**
- Single data model? **No**, each service has its own model. DTOs will be used for inter-service communication. -> **VIOLATION** (see Complexity Tracking)
- Avoiding patterns? **Yes**, starting simple.

**Architecture**:
- EVERY feature as library? **Yes**, each microservice acts as a self-contained library/feature.
- Libraries listed: **auth-service, user-service, event-service, qr-code-service, attendance-service, notification-service**
- CLI per library: **No**, this is a deviation. Will be considered for admin/debug functions.
- Library docs: **No**, deviation. README per service will be created.

**Testing (NON-NEGOTIABLE)**:
- RED-GREEN-Refactor cycle enforced? **Yes**, this will be a mandatory part of the workflow.
- Git commits show tests before implementation? **Yes**, this will be enforced during code review.
- Order: Contract→Integration→E2E→Unit strictly followed? **Yes**.
- Real dependencies used? **Yes**, integration tests will run against a real PostgreSQL DB via Docker.
- Integration tests for: new libraries, contract changes, shared schemas? **Yes**.
- FORBIDDEN: Implementation before test, skipping RED phase. **Yes**.

**Observability**:
- Structured logging included? **Yes**, will be implemented in each Go service.
- Frontend logs → backend? **Yes**, a logging endpoint will be provided.
- Error context sufficient? **Yes**, will include request IDs for tracing.

**Versioning**:
- Version number assigned? **0.1.0**
- BUILD increments on every change? **No**, will increment MINOR for new features, PATCH for fixes.
- Breaking changes handled? **Yes**, via API versioning (e.g., /v1/, /v2/).

## Project Structure

### Documentation (this feature)
```
specs/001-initialize-project/
├── plan.md              # This file (/plan command output)
├── research.md          # Phase 0 output (/plan command)
├── data-model.md        # Phase 1 output (/plan command)
├── quickstart.md        # Phase 1 output (/plan command)
├── contracts/           # Phase 1 output (/plan command)
└── tasks.md             # Phase 2 output (/tasks command - NOT created by /plan)
```

### Source Code (repository root)
```
# Option 2: Web application (when "frontend" + "backend" detected)
backend/
├── auth-service/
├── user-service/
├── event-service/
├── qr-code-service/
├── attendance-service/
└── notification-service/

frontend/
├── src/
│   ├── components/
│   ├── pages/
│   └── services/
└── tests/
```

**Structure Decision**: **Option 2: Web application** is chosen to separate the frontend from the backend microservices, as required by the specification.

## Phase 0: Outline & Research
1. **Extract unknowns from Technical Context** above:
   - Research task: Define specific performance goals (e.g., p99 latency, requests/sec) for the SportEase system under expected load.
   - Research task: Estimate the scale and scope, focusing on the number of concurrent users during a large-scale school event.

2. **Consolidate findings** in `research.md`.

## Phase 1: Design & Contracts
*Prerequisites: research.md complete*

1. **Extract entities from spec** → `data-model.md`:
   - Define schemas for User, Role, Class, Event, Team, Match, QR Code, Attendance Record.
2. **Generate API contracts** from functional requirements → `/contracts/`:
   - Create OpenAPI (v3) specifications for each microservice (auth, user, event, etc.).
3. **Generate contract tests** from contracts:
   - Create failing tests for each endpoint defined in the OpenAPI specs.
4. **Extract test scenarios** from user stories → integration tests:
   - Implement failing integration tests for the 3 Acceptance Scenarios in `spec.md`.
5. **Create `quickstart.md`**:
   - Document the steps to run the full application stack (`docker-compose up`) and test the primary user stories.

## Phase 2: Task Planning Approach
*This section describes what the /tasks command will do - DO NOT execute during /plan*

**Task Generation Strategy**:
- Load `/templates/tasks-template.md` as base.
- Generate tasks from Phase 1 design docs (contracts, data model, quickstart).
- Each contract endpoint → contract test task [P] & implementation task.
- Each entity → model creation task [P].
- Each user story → integration test task & implementation tasks to make it pass.

**Ordering Strategy**:
- TDD order: Tests before implementation.
- Dependency order: Backend services (DB, models, services, API) before frontend components.

**Estimated Output**: ~40-50 tasks in tasks.md.

## Phase 3+: Future Implementation
*These phases are beyond the scope of the /plan command*

**Phase 3**: Task execution (/tasks command creates tasks.md)
**Phase 4**: Implementation (execute tasks.md)
**Phase 5**: Validation (run tests, execute quickstart.md)

## Complexity Tracking
*Fill ONLY if Constitution Check has violations that must be justified*

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| **Multiple Projects (>3)** | The feature spec explicitly requires a "マイクロサービスベースのシステム" (microservice-based system). | A monolithic single-project structure would not meet this core architectural requirement. |
| **Multiple Data Models / DTOs** | In a microservice architecture, each service must own its data to be truly independent. DTOs are necessary for communication between services. | A single, shared data model would create tight coupling between services, defeating the purpose of a microservice architecture. |

## Progress Tracking
*This checklist is updated during execution flow*

**Phase Status**:
- [X] Phase 0: Research complete (/plan command)
- [ ] Phase 1: Design complete (/plan command)
- [ ] Phase 2: Task planning complete (/plan command - describe approach only)
- [ ] Phase 3: Tasks generated (/tasks command)
- [ ] Phase 4: Implementation complete
- [ ] Phase 5: Validation passed

**Gate Status**:
- [X] Initial Constitution Check: PASS (with justified violations)
- [ ] Post-Design Constitution Check: PENDING
- [X] All NEEDS CLARIFICATION resolved: PASS
- [X] Complexity deviations documented

---
*Based on Constitution v2.1.1 (inferred from plan template) - See `/memory/constitution.md`*
