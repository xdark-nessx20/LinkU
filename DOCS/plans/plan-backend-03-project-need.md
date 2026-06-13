# Implementation Plan: Backend — Gestión de Proyectos y Necesidades

**Date**: 2026-06-13
**Spec**: `DOCS/specs/spec-backend-03-project-need.md`

## Summary

Implementar CRUD de proyectos/necesidades vinculados a un usuario responsable, con toggle activo/inactivo, endpoint JSON para directorio con filtros, y validación de campos obligatorios.

## Technical Context

**Language/Version**: Java 21
**Primary Dependencies**: Spring Boot 3.x, Spring Data JPA, Lombok, Thymeleaf
**Storage**: PostgreSQL (entidad Project, @ManyToOne a User, @ManyToMany a SkillCategory)
**Testing**: JUnit 5, Mockito, Spring Boot Test, H2
**Target Platform**: Server (Spring Boot embedded Tomcat)
**Project Type**: web (monolith with layers)

## Project Structure

```text
src/main/java/com/unimag/match/
├── controller/
│   ├── ProjectController.java        # GET/POST HTML pages for project CRUD
│   └── ProjectApiController.java     # @RestController: GET /api/projects (JSON directory)
├── model/
│   ├── Project.java                  # @Entity: ownerId (FK to User), name, description, requiredSkills (@ManyToMany SkillCategory), ...
│   ├── User.java
│   └── SkillCategory.java
├── repository/
│   └── ProjectRepository.java        # findByOwnerId, findAllByIsActiveTrue with Specification
├── service/
│   └── ProjectService.java           # create, update, toggleActive
└── dto/
    ├── ProjectForm.java              # Form backing object
    └── ProjectDto.java               # JSON response DTO

src/main/resources/
└── templates/
    └── project/
        ├── create.html
        ├── edit.html
        ├── view.html
        └── list.html
```

## Phase 1: Setup

- [ ] T001 Verify dependencies and existing entities (User, SkillCategory) are accessible
- [ ] T002 Create Project entity with all fields per spec: owner (@ManyToOne User), name (@NotBlank), description (@NotBlank), requiredSkills (@ManyToMany SkillCategory), roleType, relatedFaculty, phase, expectedAvailability, mainObjective, expectedOutput, isActive (boolean, default true), createdAt, updatedAt
- [ ] T003 Create ProjectRepository with findByOwnerId, findByIsActiveTrue, JpaSpecificationExecutor
- [ ] T004 Create ProjectForm DTO with validation

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

- [ ] T009 [US1] Implement ProjectController.showCreate (GET) — render create.html
- [ ] T010 [US1] Implement ProjectController.create (POST) — validate, set owner from authenticated user, call ProjectService
- [ ] T011 [US1] Implement ProjectService.create
- [ ] T012 [US1] Create templates/project/create.html with form

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

- [ ] T017 [US2] Implement ProjectController.showEdit (GET) — verify ownership
- [ ] T018 [US2] Implement ProjectController.update (POST) — validate, update
- [ ] T019 [US2] Implement ProjectController.toggleActive (POST) — toggle and redirect
- [ ] T020 [US2] Implement ProjectService.update, toggleActive
- [ ] T021 [US2] Create templates/project/edit.html

---

## Phase 4: User Story 3 — Visualizar y filtrar proyectos (P2)

**Goal**: Directorio HTML con proyectos activos y endpoint JSON para filtros dinámicos.

**Independent Test**: GET /projects → HTML con proyectos activos. GET /api/projects?faculty=X&skill=Y → JSON filtrado.

### Tests for User Story 3

- [ ] T022 [US3] Integration test: GET /projects → HTML with only active projects
- [ ] T023 [US3] Integration test: GET /api/projects?skill=Python → JSON filtered by skill
- [ ] T024 [US3] Unit test: ProjectRepository with Specification returns filtered results

### Implementation for User Story 3

- [ ] T025 [US3] Implement ProjectController.listAll (GET) — render list.html with active projects
- [ ] T026 [US3] Implement ProjectController.view (GET /project/{id}) — render view.html
- [ ] T027 [US3] Implement ProjectApiController (JSON): GET /api/projects with filters (faculty, skill, phase)
- [ ] T028 [US3] Implement ProjectRepository with dynamic Specification based on request params
- [ ] T029 [US3] Create templates/project/list.html, view.html

---

## Dependencies & Execution Order

- **All phases**: Depend on plan-backend-01-auth (User entity, authentication). Also use SkillCategory from plan-backend-02.
- **US1 → US2**: Sequential (project must exist to edit).
- **US3**: After US1 (needs projects to list).

---

## Notes

- Solo usuarios con rol RESPONSABLE pueden crear/editar proyectos. Verificar en SecurityConfig (role-based access).
- isActive=true por defecto; proyectos inactivos excluidos de matching engine (plan-backend-04) y directorio.
- updatedAt se actualiza automáticamente en cada update (JPA @PreUpdate).
