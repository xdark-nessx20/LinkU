# Implementation Plan: Backend — Gestión de Perfiles de Estudiante

**Date**: 2026-06-13
**Spec**: `DOCS/specs/spec-backend-02-student-profile.md`

## Summary

Implementar CRUD de perfiles de estudiante vinculados 1:1 al User, con campos obligatorios y opcionales, marcado automático de completitud, endpoint JSON para directorio con filtros, y asociación de habilidades a categorías.

## Technical Context

**Language/Version**: Java 21
**Primary Dependencies**: Spring Boot 3.x, Spring Data JPA, Lombok, Thymeleaf
**Storage**: PostgreSQL (entidades StudentProfile, SkillCategory, tablas de unión @ManyToMany)
**Testing**: JUnit 5, Mockito, Spring Boot Test, H2
**Target Platform**: Server (Spring Boot embedded Tomcat)
**Project Type**: web (monolith with layers)

## Project Structure

```text
src/main/java/com/unimag/match/
├── config/
│   └── SecurityConfig.java
├── controller/
│   ├── ProfileController.java        # GET/POST profile HTML pages
│   └── ProfileApiController.java     # @RestController: GET /api/profiles (JSON directory)
├── model/
│   ├── User.java
│   ├── StudentProfile.java           # @Entity: userId (FK unique), fullName, faculty, program, semester, isComplete, ...
│   └── SkillCategory.java            # @Entity: name, description
├── repository/
│   ├── StudentProfileRepository.java # findByUserId, findAll with Specification filters
│   └── SkillCategoryRepository.java
├── service/
│   ├── ProfileService.java           # create, update, getByUser, markComplete
│   └── SkillCategoryService.java     # Seed catalog
└── dto/
    ├── ProfileForm.java              # Form backing object
    └── ProfileDto.java               # JSON response DTO

src/main/resources/
├── templates/
│   └── profile/
│       ├── create.html
│       ├── edit.html
│       └── view.html
└── data.sql                          # Seed data: 8 SkillCategory rows
```

## Phase 1: Setup

- [ ] T001 Verify Spring Boot project exists and dependencies are configured
- [ ] T002 Create SkillCategory entity and SkillCategoryRepository
- [ ] T003 Create data.sql with 8 seed categories: Tecnología y datos, Comunicación y divulgación, Investigación, Diseño y creatividad, Emprendimiento y gestión, Bienestar y sociedad, Ambiente y territorio, Cultura e identidad

---

## Phase 2: Foundational

- [ ] T004 Create StudentProfile entity: user_id (FK, unique, @OneToOne to User), fullName, faculty, program, semester, skills (@ManyToMany to SkillCategory with join table), interests (@ElementCollection or @ManyToMany), priorExperience, tools, availability, preferredRole, isComplete (boolean, default false), createdAt, updatedAt
- [ ] T005 Create StudentProfileRepository: findByUserId, findAll with JpaSpecificationExecutor for filtering
- [ ] T006 Create ProfileForm DTO with validation (@NotBlank on fullName, faculty, program)
- [ ] T007 Create ProfileDto for JSON directory responses (excludes userId, includes faculty, program, skills, interests)

---

## Phase 3: User Story 1 — Crear perfil (P1)

**Goal**: Estudiante autenticado crea su perfil. Un usuario = un perfil.

**Independent Test**: POST /profile/create con datos completos → StudentProfile persistido con isComplete=true. POST sin consentimiento/sin campos obligatorios → rechazo. Segundo intento de create → redirect a edit.

### Tests for User Story 1

- [ ] T008 [US1] Unit test: ProfileService.create with all fields → isComplete=true, skills linked to SkillCategory
- [ ] T009 [US1] Unit test: ProfileService.create with only mandatory fields → isComplete=false
- [ ] T010 [US1] Unit test: ProfileService.create when profile already exists → throws DuplicateProfileException
- [ ] T011 [US1] Integration test: GET /profile/create → 200 HTML form; POST /profile/create → 302 redirect

### Implementation for User Story 1

- [ ] T012 [US1] Implement ProfileController.showCreate (GET) — render create.html with SkillCategory list
- [ ] T013 [US1] Implement ProfileController.create (POST) — validate, call ProfileService, redirect to view
- [ ] T014 [US1] Implement ProfileService.create: check no existing profile, validate fields, set isComplete if all fields filled, associate skills via SkillCategory
- [ ] T015 [US1] Create templates/profile/create.html with form and skill category checkboxes

---

## Phase 4: User Story 2 — Editar perfil (P1)

**Goal**: Estudiante edita cualquier campo de su perfil existente, reflejando cambios inmediatamente.

**Independent Test**: GET /profile/edit → formulario pre-rellenado. POST cambios → perfil actualizado, updatedAt modificado. Perfil pasa de incompleto a completo.

### Tests for User Story 2

- [ ] T016 [US2] Unit test: ProfileService.update changes skills → new skills reflected, updatedAt changed
- [ ] T017 [US2] Unit test: ProfileService.update completes all fields → isComplete changes from false to true
- [ ] T018 [US2] Integration test: GET /profile/edit → 200 with populated form

### Implementation for User Story 2

- [ ] T019 [US2] Implement ProfileController.showEdit (GET) — load profile, render edit.html
- [ ] T020 [US2] Implement ProfileController.update (POST) — validate, call ProfileService.update
- [ ] T021 [US2] Implement ProfileService.update: check fields, re-evaluate isComplete, set updatedAt
- [ ] T022 [US2] Create templates/profile/edit.html (reuse form from create)

---

## Phase 5: User Story 3 — Visualizar perfil y directorio (P2)

**Goal**: Ver perfil propio (HTML) y directorio con filtros (JSON endpoint).

**Independent Test**: GET /profile → HTML con datos del perfil. GET /api/profiles?faculty=X&skill=Y → JSON paginado.

### Tests for User Story 3

- [ ] T023 [US3] Integration test: GET /profile → 200 with profile data in model
- [ ] T024 [US3] Integration test: GET /api/profiles?faculty=Ingeniería → JSON array with matching profiles
- [ ] T025 [US3] Integration test: GET /api/profiles?skill=Python → JSON filtered results
- [ ] T026 [US3] Integration test: GET /api/profiles without auth → 401/403

### Implementation for User Story 3

- [ ] T027 [US3] Implement ProfileController.view (GET) — load and render profile in view.html
- [ ] T028 [US3] Implement ProfileApiController (JSON): GET /api/profiles with @RequestParam faculty, program, skill, page
- [ ] T029 [US3] Implement StudentProfileRepository with Specification for dynamic filtering
- [ ] T030 [US3] Create templates/profile/view.html
- [ ] T031 [US3] Implement FR-PROF-007: mostrar mensaje en view.html si isComplete=false

---

## Dependencies & Execution Order

- **Phase 1-2**: Depends on plan-backend-01-auth (User entity must exist)
- **US1 → US2**: US1 must be done first (profile must exist to edit)
- **US3**: After US1 (needs profiles to list), parallel with US2

---

## Notes

- SkillCategory seed data loaded via data.sql on startup (spring.sql.init.mode=always for dev)
- [NEEDS CLARIFICATION]: Taxonomía de habilidades — se asume 8 categorías fijas del documento UNIMAG Match como seed data. Si se requiere administración dinámica, se necesita UI de administración (fuera de MVP).
- ProfileDto usado solo para el endpoint JSON del directorio; la vista HTML usa el modelo Thymeleaf directo.
