# Implementation Plan: Backend — Autenticación y Gestión de Usuarios

**Date**: 2026-06-13
**Spec**: `DOCS/specs/spec-backend-01-auth.md`

## Summary

Implementar registro de usuarios con consentimiento informado, inicio/cierre de sesión con Spring Security, recuperación de contraseña, y eliminación de cuenta con borrado de datos asociados. Este módulo es el prerrequisito para todos los demás.

## Technical Context

**Language/Version**: Java 21
**Primary Dependencies**: Spring Boot 3.x, Spring Security, Spring Data R2DBC, Lombok, Flyway
**Storage**: PostgreSQL (entidad User, sin sesiones server-side (JWT stateless))
**Testing**: JUnit 5, Mockito, Spring Boot Test, H2 (tests)
**Target Platform**: Server (Spring Boot WebFlux on Netty, one port)
**Project Type**: web (hexagonal with ports & adapters)

## Project Structure

```text
src/main/java/com/unimag/match/
├── domain/
│   ├── model/
│   │   └── User.java                    # Domain entity: email, passwordHash, role, consentGiven, consentDate
│   └── port/
│       └── UserRepository.java          # Port interface: findByEmail, existsByEmail
├── application/
│   ├── service/
│   │   └── UserService.java             # Use case: register, authenticate, deleteAccount
│   └── dto/
│       └── RegistrationForm.java        # Form backing object with validation
└── infrastructure/
    ├── config/
    │   └── SecurityConfig.java          # SecurityWebFilterChain, ReactiveUserDetailsService
    ├── persistence/
    │   └── R2dbcUserRepository.java     # R2DBC impl of UserRepository port
    └── web/
        └── AuthRestController.java      # @RestController: POST /api/auth/*

src/main/resources/
├── db/migration/
│   └── V1__create_users.sql             # Flyway migration
├── application.yml

src/test/java/com/unimag/match/
└── ...
```

## Phase 1: Setup (Shared Infrastructure)

- [ ] T001 Create Spring Boot project with Gradle (Spring WebFlux, Security, Data R2DBC, PostgreSQL, Lombok, Validation, Flyway)
- [ ] T002 Configure application.properties (R2DBC PostgreSQL connection, server port, CORS, Flyway)
- [ ] T003 Configure SecurityConfig as SecurityWebFilterChain with JWT token generation and validation, and BCryptPasswordEncoder bean

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: User entity and repository that ALL other modules depend on.

- [ ] T004 Create V1__create_users.sql Flyway migration (email, password_hash, role, consent_given, consent_date, created_at, updated_at)
- [ ] T005 Create User domain entity in domain/model/User.java (pure domain object)
- [ ] T006 Create UserRepository port interface in domain/port/UserRepository.java (findByEmail, existsByEmail)
- [ ] T007 Create R2dbcUserRepository adapter in infrastructure/persistence/R2dbcUserRepository.java
- [ ] T008 Configure SecurityConfig to use ReactiveUserDetailsService backed by UserRepository
- [ ] T009 Create RegistrationForm DTO in application/dto/RegistrationForm.java with validation (@Email, @NotBlank, @Size(min=8), consentGiven must be true)
- [ ] T010 Create AuthRestController basic skeleton in infrastructure/web/AuthRestController.java

---

## Phase 3: User Story 1 — Registro de cuenta con consentimiento (P1)

**Goal**: Visitante puede crear cuenta con email, password, rol, y consentimiento obligatorio.

**Independent Test**: POST /register con datos válidos → usuario persiste en DB con consentGiven=true y BCryptPasswordEncoder. Sin consentimiento → rechazo.

### Tests for User Story 1

- [ ] T010 [US1] Unit test: UserService.register with valid RegistrationForm → User persisted, password hashed
- [ ] T011 [US1] Unit test: UserService.register with consentGiven=false → throws ValidationException
- [ ] T012 [US1] Unit test: UserService.register with duplicate email → throws DuplicateEmailException
- [ ] T013 [US1] Integration test: POST /register valid data → 302 redirect to dashboard, User in DB

### Implementation for User Story 1

- [ ] T014 [US1] Implement AuthRestController.register (POST /api/auth/register): valida body JSON, returns Mono<ResponseEntity<AuthResponse>> con JWT
- [ ] T015 [US1] Implement UserService.register: validate, hash password with BCryptPasswordEncoder, persist User
- [ ] T016 [US1] Add server-side validation for duplicate email and consent checkbox
- [ ] T017 [US1] Add validation error responses in JSON format (400 Bad Request con campo "errors")

---

## Phase 4: User Story 2 — Inicio y cierre de sesión (P1)

**Goal**: Usuario inicia sesión con email/password, accede a rutas protegidas, cierra sesión.

**Independent Test**: POST /login con credenciales válidas → sesión iniciada, acceso a /perfil. POST /login credenciales inválidas → mensaje genérico. GET /logout → sesión destruida.

### Tests for User Story 2

- [ ] T018 [US2] Integration test: POST /login valid credentials → 302, session cookie set
- [ ] T019 [US2] Integration test: GET /perfil without session → 302 redirect to /login
- [ ] T020 [US2] Integration test: POST /login invalid credentials → error message, no session
- [ ] T021 [US2] Integration test: GET /logout → session invalidated, redirect to /

### Implementation for User Story 2

- [ ] T022 [US2] Configure SecurityWebFilterChain: disable formLogin, enable stateless JWT auth, configure JWT filter
- [ ] T023 [US2] Implement ReactiveUserDetailsService loading User from UserRepository port for JWT authentication
- [ ] T024 [US2] Implement POST /api/auth/login: authenticate credentials, generate JWT, return AuthResponse
- [ ] T025 [US2] Implement POST /api/auth/logout: invalidate JWT (add to deny list or blacklist)
- [ ] T026 [US2] Configure CORS and CSRF (disable CSRF for REST API, configure CORS for React dev server)

---

## Phase 5: User Story 3 — Recuperación de contraseña (P2)

**Goal**: Usuario puede solicitar restablecimiento de contraseña.

**Independent Test**: Solicitar restablecimiento para email existente → sistema procesa (implementación concreta depende de [NEEDS CLARIFICATION: infraestructura de email]).

### Implementation for User Story 3

- [ ] T027 [US3] Create password-recovery.html template (email field)
- [ ] T028 [US3] Implement AuthController.forgotPassword (POST) — muestra mensaje genérico
- [ ] T029 [US3] [NEEDS CLARIFICATION: implementar envío de email con token o mecanismo alternativo según decisión de infraestructura]

---

## Phase 6: User Story 4 — Eliminación de cuenta (P1)

**Goal**: Usuario autenticado puede eliminar su cuenta y todos sus datos asociados.

**Independent Test**: POST /delete-account → usuario, perfil, matches y notificaciones eliminados de DB.

### Implementation for User Story 4

- [ ] T030 [US4] Implement AuthController.deleteAccount (POST) con confirmación
- [ ] T031 [US4] Implement UserService.deleteAccount: reactive transaction, cascada a StudentProfile, Matches, Notifications
- [ ] T032 [US4] Add confirmation page/template before delete
- [ ] T033 [US4] Test que email queda liberado para nuevo registro después de delete

---

## Dependencies & Execution Order

- **Phase 1**: No dependencies
- **Phase 2**: Depends on Phase 1
- **Phase 3 (US1)**: Depends on Phase 2
- **Phase 4 (US2)**: Depends on Phase 2
- **Phase 5 (US3)**: Depends on Phase 2, parallel with US1/US2
- **Phase 6 (US4)**: Depends on Phase 2, parallel with other stories

---

## Notes

- [US1] label maps task to User Story 1 for traceability
- Each user story independently testable
- BCryptPasswordEncoder bean in SecurityConfig for password hashing and JWT login verification
- Errores de API retornados como JSON con códigos HTTP apropiados (400, 401, 409, etc.)
- Flyway migrations handle schema creation (no data.sql for DDL)
- [NEEDS CLARIFICATION]: US3 depende de decisión sobre infraestructura de email
- Clean code: meaningful names, small methods, SRP. No dead code, no commented-out code.
