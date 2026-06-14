# Implementation Plan: Backend — Gestión de Proyectos y Necesidades

**Date**: 2026-06-13
**Spec**: `DOCS/specs/spec-backend-03-project-need.md`

## Summary

Implementar CRUD de proyectos/necesidades vinculados a un usuario responsable, con toggle activo/inactivo, endpoint JSON para directorio con filtros, y validación de campos obligatorios.

## Technical Context

**Language/Version**: Java 21
**Primary Dependencies**: Spring Boot 3.x, Spring Data R2DBC, Lombok, Flyway
**Storage**: PostgreSQL (entidad Project, join table para skills)
**Testing**: JUnit 5, Mockito, Spring Boot Test, H2
**Target Platform**: Server (Spring Boot WebFlux on Netty)
**Project Type**: web (hexagonal with ports & adapters)

## Project Structure

```text
src/main/java/com/unimag/match/
├── domain/
│   ├── model/
│   │   ├── Project.java               # Domain entity
│   │   ├── User.java
│   │   └── SkillCategory.java
│   └── port/
│       └── ProjectRepository.java     # Port interface
├── application/
│   ├── service/
│   │   └── ProjectService.java        # Use case: create, update, toggleActive
│   └── dto/
│       ├── ProjectRequest.java         # Create/Update JSON request DTO
│       └── ProjectResponse.java        # JSON response DTO
└── infrastructure/
    ├── persistence/
    │   └── R2dbcProjectRepository.java  # R2DBC adapter
    └── web/
        └── ProjectRestController.java  # @RestController: GET/POST/PUT /api/projects/*

src/main/resources/
├── db/migration/
│   └── V4__create_projects.sql        # Flyway migration
```

## Phase 1: Setup

- [ ] T001 Verify dependencies and existing entities (User, SkillCategory) are accessible
- [ ] T002 Create V4__create_projects.sql Flyway migration (owner_id FK to users, name, description, required_skills via join table, role_type, related_faculty, phase, expected_availability, main_objective, expected_output, is_active, created_at, updated_at)
- [ ] T003 Create Project domain entity in domain/model/Project.java
- [ ] T004 Create ProjectRepository port interface in domain/port/ProjectRepository.java (findByOwnerId, findByIsActiveTrue with filtering)
- [ ] T005 Create R2dbcProjectRepository adapter in infrastructure/persistence/
- [ ] T006 Create ProjectRequest/ProjectResponse DTOs for JSON in application/dto/ with validation

---

## Phase 2: User Story 1 — Registrar proyecto/necesidad (P1)

**Goal**: Responsable crea proyecto con campos obligatorios y opcionales.

**Independent Test**: POST /project/create con datos → Project persistido, isActive=true. Campos obligatorios vacíos → rechazo.

### Tests for User Story 1

- [ ] T005 [US1] Unit test: ProjectService.create with valid data → Project persisted, owner set
- [ ] T006 [US1] Unit test: ProjectService.create with empty name → throws ValidationException
- [ ] T007 [US1] Unit test: ProjectService.create without requiredSkills → allowed (skills are optional)
- [ ] T008 [US1] Integration test: POST /project/create → 302 redirect, project in DB

### Implementation for User Story 1

- [ ] T009 [US1] Implement ProjectRestController.create (POST /api/projects) — validate, set owner from authenticated user, call ProjectService
- [ ] T010 [US1] Implement ProjectService.create

---

## Phase 3: User Story 2 — Editar y gestionar proyectos (P1)

**Goal**: Responsable edita su proyecto y lo marca activo/inactivo. Solo el owner puede editar.

**Independent Test**: GET /project/{id}/edit → 200 si es owner. POST cambios → proyecto actualizado. POST /project/{id}/toggle → isActive cambia.

### Tests for User Story 2

- [ ] T013 [US2] Unit test: ProjectService.update changes skills → new skills reflected
- [ ] T014 [US2] Unit test: ProjectService.toggleActive true→false → isActive=false
- [ ] T015 [US2] Unit test: ProjectService.toggleActive false→true → isActive=true
- [ ] T016 [US2] Integration test: GET /project/{id}/edit by non-owner → 403

### Implementation for User Story 2

- [ ] T017 [US2] Implement ProjectRestController.update (PUT /api/projects/{id}) — verify ownership, validate, update
- [ ] T018 [US2] Implement ProjectRestController.toggleActive (PATCH /api/projects/{id}/active) — toggle and return updated project
- [ ] T019 [US2] Implement ProjectService.update, toggleActive

---

## Phase 4: User Story 3 — Visualizar y filtrar proyectos (P2)

**Goal**: Directorio JSON con proyectos activos y filtros via query params en GET.

**Independent Test**: GET /api/projects → JSON con proyectos activos. GET /api/projects?faculty=X → JSON con resultados filtrados.

### Tests for User Story 3

- [ ] T022 [US3] Integration test: GET /api/projects → JSON with only active projects
- [ ] T023 [US3] Integration test: GET /api/projects?skill=Python → JSON filtered by skill
- [ ] T024 [US3] Unit test: ProjectRepository with filter params returns filtered results

### Implementation for User Story 3

- [ ] T025 [US3] Implement ProjectRestController.list (GET /api/projects?faculty=&skill=&isActive=true) — query with optional filters, return JSON
- [ ] T026 [US3] Implement ProjectRestController.getById (GET /api/projects/{id}) — return JSON

---

## Dependencies & Execution Order

- **All phases**: Depend on plan-backend-01-auth (User entity, authentication). Also use SkillCategory from plan-backend-02.
- **US1 → US2**: Sequential (project must exist to edit).
- **US3**: After US1 (needs projects to list).

---

## Notes

- Solo usuarios con rol RESPONSABLE (o rol híbrido estudiante+responsable) pueden crear/editar proyectos. Verificar en SecurityWebFilterChain (role-based access).
- Usuarios con rol "estudiante" pueden crear proyectos cambiando su rol a "responsable" o mediante un rol híbrido con ambos permisos.
- Sin límite de creación de proyectos por usuario.
- isActive=true por defecto; proyectos inactivos excluidos de matching engine (plan-backend-04) y directorio.
- updatedAt se actualiza automáticamente en cada update (via R2DBC audit o manual).
- Directorio de proyectos: REST endpoint GET /api/projects con query params. Respuesta JSON.
- Clean code: meaningful names, small methods, SRP. No dead code, no commented-out code.
