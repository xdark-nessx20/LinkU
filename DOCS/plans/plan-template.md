# Implementation Plan: [FEATURE]

**Date**: [DATE] 
**Spec**: [link]

## Summary

[Extract from feature spec: primary requirement + technical approach from research]

## Technical Context

<!--
  ACTION REQUIRED: Replace the content in this section with the technical details
  for the project. The structure here is presented in advisory capacity to guide
  the iteration process.
-->

**Language/Version**: [e.g., Python 3.11, Swift 5.9, Rust 1.75 or NEEDS CLARIFICATION]  
**Primary Dependencies**: [e.g., FastAPI, UIKit, LLVM or NEEDS CLARIFICATION]  
**Storage**: [if applicable, e.g., PostgreSQL, CoreData, files or N/A]  
**Testing**: [e.g., pytest, XCTest, cargo test or NEEDS CLARIFICATION]  
**Target Platform**: [e.g., Linux server, iOS 15+, WASM or NEEDS CLARIFICATION]
**Project Type**: [single/web/mobile - determines source structure]  
**Performance Goals**: [domain-specific, e.g., 1000 req/s, 10k lines/sec, 60 fps or NEEDS CLARIFICATION]  
**Constraints**: [domain-specific, e.g., <200ms p95, <100MB memory, offline-capable or NEEDS CLARIFICATION]  
**Scale/Scope**: [domain-specific, e.g., 10k users, 1M LOC, 50 screens or NEEDS CLARIFICATION]

## Project Structure

### Documentation (this feature)

```text
specs/[feature]/
├── plan.md              # This file 
└── spec.md             # Phase 2 output (/speckit.tasks command - NOT created by /speckit.plan)
```

### Source Code (repository root)
<!--
  ACTION REQUIRED: Replace the placeholder tree below with the concrete layout
  for this feature. Delete unused options and expand the chosen structure with
  real paths (e.g., apps/admin, packages/something). The delivered plan must
  not include Option labels.
-->

```text
backend/
├── build.gradle.kts
├── src/main/java/com/unimag/match/
│   ├── domain/
│   │   ├── model/           # Domain entities (no framework annotations)
│   │   ├── service/         # Domain service interfaces
│   │   └── port/            # Port interfaces (repositories, etc.)
│   ├── application/
│   │   ├── service/         # Use case implementations
│   │   └── dto/             # DTOs, request/response objects
│   └── infrastructure/
│       ├── config/          # SecurityWebFilterChain, JWT, CORS, Flyway
│       ├── persistence/     # R2DBC repository implementations
│       └── web/             # @RestController (Mono<ResponseEntity<T>>)
├── src/main/resources/
│   ├── db/migration/        # Flyway SQL migrations (V1__, V2__, ...)
│   ├── static/              # React SPA build output
│   └── application.properties
└── src/test/

frontend/
├── src/
│   ├── components/
│   ├── pages/
│   └── services/
└── tests/
```

**Structure Decision**: [Document the selected structure and reference the real
directories captured above]


<!-- 
  ============================================================================
  IMPORTANT: The tasks below are SAMPLE TASKS for illustration purposes only.
  
  You MUST replace these with actual tasks based on:
  - User stories from spec.md
  - Feature requirements from this file
  - Entities required for the use case
  - Endpoints required
  
  DO NOT keep these sample tasks.
  ============================================================================
-->

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization and basic structure

- [ ] T001 Create project structure per implementation plan
- [ ] T002 Initialize [language] project with [framework] dependencies
- [ ] T003 Configure linting and formatting tools

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core infrastructure that MUST be complete before ANY user story can be implemented

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

Examples of foundational tasks (adjust based on your project):

- [ ] T004 Setup database schema with Flyway migration scripts
- [ ] T005 Implement authentication/authorization framework (JWT + SecurityWebFilterChain)
- [ ] T006 Setup REST API controller structure with global error handling
- [ ] T007 Create base domain entities and port interfaces that all stories depend on
- [ ] T008 Configure CORS, logging, and validation infrastructure
- [ ] T009 Setup environment configuration management (application.properties)

**Checkpoint**: Foundation ready - user story implementation can now begin in parallel

---

## Phase 3: User Story 1 - [Title] (Priority: P1) 

**Goal**: [Brief description of what this story delivers]

**Independent Test**: [How to verify this story works on its own]

### Tests for User Story 1 

- [ ] T010 [P] [US1] Contract test for [use case] in test/domain/[usecase]Test.java
- [ ] T011 [P] [US1] Integration test for [user journey] in test/infrastructure/web/[controller]Test.java

### Implementation for User Story 1

- [ ] T012 [P] [US1] Create [Entity1] domain model in domain/model/[entity1].java
- [ ] T013 [P] [US1] Create [Entity1] port interface in domain/port/[entity1]Repository.java
- [ ] T014 [US1] Implement [Service] in application/service/[service].java (depends on T012, T013)
- [ ] T015 [US1] Implement [RestController] in infrastructure/web/[controller].java (Mono<ResponseEntity<T>>)
- [ ] T016 [US1] Create Flyway migration V[N]__create_[table].sql
- [ ] T017 [US1] Add validation, error handling, and API error responses

**Checkpoint**: At this point, User Story 1 should be fully functional and testable independently

---

## Phase 4: User Story 2 - [Title] (Priority: P2)

**Goal**: [Brief description of what this story delivers]

**Independent Test**: [How to verify this story works on its own]

### Tests for User Story 2 

- [ ] T018 [P] [US2] Contract test for [use case] in test/domain/[usecase]Test.java
- [ ] T019 [P] [US2] Integration test for [user journey] in test/infrastructure/web/[controller]Test.java

### Implementation for User Story 2

- [ ] T020 [P] [US2] Create [Entity] domain model in domain/model/[entity].java
- [ ] T021 [US2] Implement [Service] in application/service/[service].java
- [ ] T022 [US2] Implement [RestController] in infrastructure/web/[controller].java
- [ ] T023 [US2] Integrate with User Story 1 components (if needed)

**Checkpoint**: At this point, User Stories 1 AND 2 should both work independently

---

## Phase 5: User Story 3 - [Title] (Priority: P3)

**Goal**: [Brief description of what this story delivers]

**Independent Test**: [How to verify this story works on its own]

### Tests for User Story 3 

- [ ] T024 [P] [US3] Contract test for [use case] in test/domain/[usecase]Test.java
- [ ] T025 [P] [US3] Integration test for [user journey] in test/infrastructure/web/[controller]Test.java

### Implementation for User Story 3

- [ ] T026 [P] [US3] Create [Entity] domain model in domain/model/[entity].java
- [ ] T027 [US3] Implement [Service] in application/service/[service].java
- [ ] T028 [US3] Implement [RestController] in infrastructure/web/[controller].java

**Checkpoint**: All user stories should now be independently functional

---

[Add more user story phases as needed, following the same pattern]

---

## Phase N: Polish & Cross-Cutting Concerns

**Purpose**: Improvements that affect multiple user stories

- [ ] TXXX Documentation updates in docs/
- [ ] TXXX Code cleanup and refactoring
- [ ] TXXX Performance optimization across all stories
- [ ] TXXX Additional unit tests in tests/unit/
- [ ] TXXX Security hardening

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion - BLOCKS all user stories
- **User Stories (Phase 3+)**: All depend on Foundational phase completion
  - User stories can then proceed in parallel (if staffed)
  - Or sequentially in priority order (P1 → P2 → P3)
- **Polish (Final Phase)**: Depends on all desired user stories being complete

### User Story Dependencies

- **User Story 1 (P1)**: Can start after Foundational (Phase 2) - No dependencies on other stories
- **User Story 2 (P2)**: Can start after Foundational (Phase 2) - May integrate with US1 but should be independently testable
- **User Story 3 (P3)**: Can start after Foundational (Phase 2) - May integrate with US1/US2 but should be independently testable

### Within Each User Story

- Port interfaces before adapters
- Domain models before application services
- Application services before REST controllers
- Core implementation before integration
- Story complete before moving to next priority
- Tests after implementation

## Notes

- [Story] label maps task to specific user story for traceability
- Each user story should be independently completable and testable
- All controllers return `Mono<ResponseEntity<T>>` (WebFlux reactive)
- No Thymeleaf, no server-rendered pages — 100% REST API
- Verify tests pass
- Commit after each task or logical group
- Stop at any checkpoint to validate story independently
- Avoid: vague tasks, same file conflicts, cross-story dependencies that break independence
