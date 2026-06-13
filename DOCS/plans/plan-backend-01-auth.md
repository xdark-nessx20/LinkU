# Implementation Plan: Backend — Autenticación y Gestión de Usuarios

**Date**: 2026-06-13
**Spec**: `DOCS/specs/spec-backend-01-auth.md`

## Summary

Implementar registro de usuarios con consentimiento informado, inicio/cierre de sesión con Spring Security, recuperación de contraseña, y eliminación de cuenta con borrado de datos asociados. Este módulo es el prerrequisito para todos los demás.

## Technical Context

**Language/Version**: Java 21
**Primary Dependencies**: Spring Boot 3.x, Spring Security, Spring Data JPA, Lombok, Thymeleaf
**Storage**: PostgreSQL (entidad User, sesiones en memoria vía HttpSession)
**Testing**: JUnit 5, Mockito, Spring Boot Test, H2 (tests)
**Target Platform**: Server (Spring Boot embedded Tomcat, one port)
**Project Type**: web (monolith with layers)

## Project Structure

```text
src/main/java/com/unimag/match/
├── config/
│   └── SecurityConfig.java
├── controller/
│   └── AuthController.java          # GET/POST login, register, logout (HTML pages)
├── model/
│   └── User.java                    # @Entity: email, passwordHash, role, consentGiven, consentDate
├── repository/
│   └── UserRepository.java          # findByEmail, existsByEmail
├── service/
│   └── UserService.java             # register, authenticate, deleteAccount
└── dto/
    └── RegistrationForm.java        # Form backing object with validation

src/main/resources/
├── templates/
│   ├── auth/
│   │   ├── login.html              # Thymeleaf
│   │   ├── register.html
│   │   └── password-recovery.html
│   └── ...
├── application.properties
└── data.sql

src/test/java/com/unimag/match/
└── ...
```

## Phase 1: Setup (Shared Infrastructure)

- [ ] T001 Create Spring Boot project with Spring Initializr (Spring Web, Security, Data JPA, PostgreSQL, Thymeleaf, Lombok, Validation)
- [ ] T002 Configure application.properties (PostgreSQL connection, server port, Thymeleaf)
- [ ] T003 Configure SecurityConfig with basic form-login and BCryptPasswordEncoder bean

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: User entity and repository that ALL other modules depend on.

- [ ] T004 Create User entity: email (unique, not null), passwordHash, role (enum ESTUDIANTE/RESPONSABLE), consentGiven, consentDate, createdAt, updatedAt
- [ ] T005 Create UserRepository: findByEmail, existsByEmail
- [ ] T006 Configure SecurityConfig to use UserRepository via UserDetailsService
- [ ] T007 Create RegistrationForm DTO with validation (@Email, @NotBlank, @Size(min=8), consentGiven must be true)
- [ ] T008 Create templates/auth/register.html (Thymeleaf form with consent checkbox)
- [ ] T009 Create templates/auth/login.html

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

- [ ] T014 [US1] Implement AuthController.showRegister (GET) and AuthController.register (POST)
- [ ] T015 [US1] Implement UserService.register: validate, hash password with BCryptPasswordEncoder, persist User
- [ ] T016 [US1] Add server-side validation for duplicate email and consent checkbox
- [ ] T017 [US1] Add success/error flash messages in Thymeleaf

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

- [ ] T022 [US2] Configure SecurityConfig: loginPage("/login"), loginProcessingUrl("/login"), defaultSuccessUrl based on role
- [ ] T023 [US2] Implement UserDetailsService loading User from UserRepository
- [ ] T024 [US2] Implement logout: logoutUrl("/logout"), invalidateHttpSession(true), deleteCookies("JSESSIONID")
- [ ] T025 [US2] Add generic error message on failed login (no email/user distinction)
- [ ] T026 [US2] Configure HTTP security: permitAll on /login, /register, /css/**, /js/**; authenticated for everything else

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
- [ ] T031 [US4] Implement UserService.deleteAccount: @Transactional, cascada a StudentProfile, Matches, Notifications
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
- BCryptPasswordEncoder bean in SecurityConfig, auto-injected
- Flash attributes for form validation errors in Thymeleaf
- [NEEDS CLARIFICATION]: US3 depende de decisión sobre infraestructura de email
