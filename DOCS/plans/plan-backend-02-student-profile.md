# Implementation Plan: Backend — Gestión de Perfiles de Estudiante

**Date**: 2026-06-13
**Spec**: `DOCS/specs/spec-backend-02-student-profile.md`

## Summary

Implementar CRUD de perfiles de estudiante vinculados 1:1 al User, con campos obligatorios y opcionales, marcado automático de completitud, endpoint JSON para directorio con filtros, y asociación de habilidades a categorías.

## Technical Context

**Language/Version**: Java 21
**Primary Dependencies**: Spring Boot 3.x, Spring Data R2DBC, Lombok, Flyway
**Storage**: PostgreSQL (entidades StudentProfile, SkillCategory, tablas de unión)
**Testing**: JUnit 5, Mockito, Spring Boot Test, H2
**Target Platform**: Server (Spring Boot WebFlux on Netty)
**Project Type**: web (hexagonal with ports & adapters)

## Project Structure

```text
src/main/java/com/unimag/match/
├── domain/
│   ├── model/
│   │   ├── StudentProfile.java         # Domain entity
│   │   └── SkillCategory.java          # Domain entity
│   └── port/
│       ├── StudentProfileRepository.java   # Port interface
│       └── SkillCategoryRepository.java    # Port interface
├── application/
│   ├── service/
│   │   ├── ProfileService.java         # Use case: create, update, getByUser, markComplete
│   │   └── SkillCategoryService.java   # Seed catalog via Flyway
│   └── dto/
│       └── ProfileForm.java            # Form backing object
└── infrastructure/
    ├── config/
    │   └── SecurityConfig.java
    ├── persistence/
    │   ├── R2dbcStudentProfileRepository.java  # R2DBC adapter
    │   └── R2dbcSkillCategoryRepository.java   # R2DBC adapter
    └── web/
        └── ProfileRestController.java  # @RestController: GET/POST/PUT /api/profiles/* (Mono<ResponseEntity<T>>)

src/main/resources/
├── db/migration/
│   ├── V2__create_skill_categories.sql  # Seed data (8 categories)
│   └── V3__create_student_profiles.sql  # StudentProfile table
└── application.properties
```

## Phase 1: Setup

- [ ] T001 Verify Spring Boot project with Gradle, WebFlux, R2DBC, Flyway dependencies
- [ ] T002 Create V2__create_skill_categories.sql Flyway migration with 8 seed categories: Tecnología y datos, Comunicación y divulgación, Investigación, Diseño y creatividad, Emprendimiento y gestión, Bienestar y sociedad, Ambiente y territorio, Cultura e identidad
- [ ] T003 Create SkillCategory port interface in domain/port/SkillCategoryRepository.java

---

## Phase 2: Foundational

- [ ] T004 Create V3__create_student_profiles.sql Flyway migration (user_id FK unique, full_name, faculty, program, semester, skills via join table, interests via join table, prior_experience, tools, availability, preferred_role, is_complete, created_at, updated_at)
- [ ] T005 Create StudentProfile domain entity in domain/model/StudentProfile.java
- [ ] T006 Create StudentProfileRepository port interface in domain/port/StudentProfileRepository.java (findByUserId, findAll with filtering)
- [ ] T007 Create R2DBC adapter implementations for both repositories
- [ ] T008 Create ProfileForm DTO in application/dto/ProfileForm.java with validation (@NotBlank on fullName, faculty, program)

---

## Phase 3: User Story 1 — Crear perfil (P1)

**Goal**: Estudiante autenticado crea su perfil. Un usuario = un perfil.

**Independent Test**: POST /api/profiles con datos completos → 201 Created, isComplete=true. POST sin campos obligatorios → 400 Bad Request. Duplicado → 409 Conflict.

### Tests for User Story 1

- [ ] T009 [US1] Unit test: ProfileService.create with all fields → isComplete=true, skills linked to SkillCategory
- [ ] T010 [US1] Unit test: ProfileService.create with only mandatory fields → isComplete=false
- [ ] T011 [US1] Unit test: ProfileService.create when profile already exists → throws DuplicateProfileException
- [ ] T012 [US1] Integration test: POST /api/profiles with valid data → 201 Created; duplicate POST → 409 Conflict

### Implementation for User Story 1

- [ ] T013 [US1] Implement ProfileRestController.create (POST /api/profiles) — validate, call ProfileService, return created profile
- [ ] T015 [US1] Implement ProfileService.create: check no existing profile, validate fields, set isComplete if all fields filled, associate skills via SkillCategory

---

## Phase 4: User Story 2 — Editar perfil (P1)

**Goal**: Estudiante edita cualquier campo de su perfil existente, reflejando cambios inmediatamente.

**Independent Test**: GET /api/profiles/me → 200 con datos del perfil. PUT /api/profiles → perfil actualizado, updatedAt modificado. Perfil pasa de incompleto a completo.

### Tests for User Story 2

- [ ] T017 [US2] Unit test: ProfileService.update changes skills → new skills reflected, updatedAt changed
- [ ] T018 [US2] Unit test: ProfileService.update completes all fields → isComplete changes from false to true
- [ ] T019 [US2] Integration test: GET /api/profiles/me → 200 with profile JSON

### Implementation for User Story 2

- [ ] T020 [US2] Implement ProfileRestController.get (GET /api/profiles/me) — load profile, return as JSON
- [ ] T021 [US2] Implement ProfileRestController.update (PUT /api/profiles) — validate, call ProfileService.update
- [ ] T022 [US2] Implement ProfileService.update: check fields, re-evaluate isComplete, set updatedAt

---

## Phase 5: User Story 3 — Visualizar perfil y directorio (P2)

**Goal**: Ver perfil propio y directorio con filtros vía REST JSON.

**Independent Test**: GET /api/profiles/me → 200 con perfil en JSON. GET /api/profiles?faculty=Ingeniería → 200 con resultados paginados.

### Tests for User Story 3

- [ ] T024 [US3] Integration test: GET /api/profiles/me → 200 with profile JSON
- [ ] T025 [US3] Integration test: GET /api/profiles?faculty=Ingeniería → 200 with paginated results
- [ ] T026 [US3] Integration test: GET /api/profiles?skill=Python → 200 with paginated results
- [ ] T027 [US3] Integration test: GET /api/profiles/me without auth → 401 Unauthorized

### Implementation for User Story 3

- [ ] T028 [US3] Implement ProfileRestController.getById (GET /api/profiles/{id}) — load and return profile as JSON
- [ ] T029 [US3] Implement ProfileRestController.list (GET /api/profiles?faculty=&skill=&program=&page=&size=) — accept query params, query repository with filtering and pagination
- [ ] T030 [US3] Implement paginated response DTOs (ProfilePageResponse)
- [ ] T031 [US3] ProfileRestController returns message field in response when isComplete=false

---

## Dependencies & Execution Order

- **Phase 1-2**: Depends on plan-backend-01-auth (User entity must exist)
- **US1 → US2**: US1 must be done first (profile must exist to edit)
- **US3**: After US1 (needs profiles to list), parallel with US2

---

## Notes

- SkillCategory seed data loaded via Flyway migration V2__create_skill_categories.sql
- [NEEDS CLARIFICATION]: Taxonomía de habilidades — se asume 8 categorías fijas del documento UNIMAG Match como seed data. Si se requiere administración dinámica, se necesita UI de administración (fuera de MVP).
- Directorio de perfiles: REST endpoint GET /api/profiles con query params. Respuesta JSON paginada.
- Clean code: meaningful names, small methods, SRP. No dead code, no commented-out code.
