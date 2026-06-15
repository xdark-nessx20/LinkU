# Implementation Plan: Backend — Gestión de Interacciones (Match)

**Date**: 2026-06-13
**Spec**: `DOCS/specs/spec-backend-05-match-interactions.md`

## Summary

Implementar el ciclo de vida del match: marcar "me interesa", aceptar/rechazar, notificaciones in-app persistidas, historial de matches por usuario. Exponer endpoints JSON para las acciones de interacción.

## Technical Context

**Language/Version**: Java 21
**Primary Dependencies**: Spring Boot 3.x, Spring Data R2DBC, Lombok, Flyway
**Storage**: PostgreSQL (entidades Match, Notification)
**Testing**: JUnit 5, Mockito, Spring Boot Test, H2
**Target Platform**: Server (Spring Boot WebFlux on Netty)
**Project Type**: web (hexagonal with ports & adapters)

## Project Structure

```text
src/main/java/com/unimag/match/
├── domain/
│   ├── model/
│   │   ├── Match.java                # Domain entity: studentId, projectId, status, initiatedBy, matchScore, createdAt, updatedAt. Unique(studentId, projectId)
│   │   └── Notification.java         # Domain entity: userId, matchId, type, message, isRead, createdAt
│   └── port/
│       ├── MatchRepository.java      # Port interface
│       └── NotificationRepository.java  # Port interface
├── application/
│   ├── service/
│   │   ├── MatchService.java         # Use case: expressInterest, accept, reject, getHistory
│   │   └── NotificationService.java  # createNotification, markAsRead, getUnreadCount
│   └── dto/
│       ├── MatchRequest.java         # Request DTO for match actions
│       └── MatchResponse.java        # Response DTO for match data
└── infrastructure/
    ├── persistence/
    │   ├── R2dbcMatchRepository.java
    │   └── R2dbcNotificationRepository.java
    └── web/
        ├── MatchRestController.java      # @RestController: POST /api/matches, POST /api/matches/{id}/accept, POST /api/matches/{id}/reject, GET /api/matches
        └── NotificationRestController.java  # @RestController: notification endpoints

src/main/resources/
├── db/migration/
│   ├── V6__create_matches.sql        # Flyway migration
│   └── V7__create_notifications.sql  # Flyway migration
```

## Clean Code Guidelines

### Naming & Style
- **Clases/Interfaces**: `PascalCase` — `MatchService`, `NotificationService`, `MatchRepository`, `NotificationRepository`, `MatchRestController`, `NotificationRestController`
- **Métodos**: `camelCase` — `expressInterest()`, `acceptMatch()`, `rejectMatch()`, `getMatchHistory()`, `createNotification()`, `markAsRead()`, `getUnreadCount()`
- **Constantes**: `UPPER_SNAKE_CASE` — `MATCH_STATUS_SUGERIDO`, `MATCH_STATUS_INTERESADO`, `MATCH_STATUS_ACEPTADO`, `MATCH_STATUS_RECHAZADO`
- **Paquetes**: `lowercase` sin guiones — `com.unimag.match.domain.model`, `com.unimag.match.application.dto`
- **DTOs**: Sufijo explícito — `MatchRequest` (request para expresar interés), `MatchResponse` (respuesta JSON con estado y datos), `NotificationResponse`

### Single Responsibility
- Cada clase debe tener UNA razón para cambiar
- **Controllers**: Solo HTTP — `MatchRestController` parsea requests de interacción, delega a `MatchService`; `NotificationRestController` maneja endpoints de notificaciones; retornan `Mono<ResponseEntity<T>>`
- **Services**: Solo lógica de negocio — `MatchService` maneja ciclo de vida del match (expresar interés, aceptar, rechazar), validación de estados; `NotificationService` maneja creación, lectura y consulta de notificaciones; nunca acceden a HTTP ni a DB directamente
- **Repositories**: Solo acceso a datos — `MatchRepository` expone `findByStudentId`, `findByProjectId`, `findByStudentAndProject`; `NotificationRepository` expone `findByUserId`, `countUnreadByUserId`
- **Domain/Entities**: Solo datos y validaciones básicas — `Match` encapsula estado, transiciones permitidas (`canAccept`, `canReject`); `Notification` encapsula tipo, mensaje, isRead
- Si una clase supera ~300 líneas, extraer responsabilidades (ej. `MatchStateValidator` separado de `MatchService`)
- Separar paquetes por dominio: `match` agrupa model, service, dto de matches; `notification` para notificaciones

### Métodos limpios
- Máximo ~30 líneas por método; extraer bloques lógicos a métodos privados con nombre descriptivo (ej. `validateStateTransition`, `determineNotificationRecipient`, `buildMatchResponse`)
- **Guard clauses al inicio**: `if (existingMatch != null) throw new DuplicateMatchException(…)` — retornar o lanzar temprano
- Un solo nivel de abstracción por método: `expressInterest` valida unicidad, crea Match, determina destinatario y crea notificación; delega cada paso a métodos privados
- Evitar más de 3 niveles de indentación; usar `Optional` para búsquedas y validaciones de estado
- Métodos de consulta (`getHistory`, `getUnreadCount`) deben ser side-effect-free; métodos de comando (`expressInterest`, `accept`, `reject`) deben documentar el efecto y las transiciones de estado

### Principios SOLID
- **S**: Single Responsibility (ver arriba)
- **O**: Abierto a extensión — `MatchStatus` como enum con comportamiento de transiciones permite agregar nuevos estados sin modificar `MatchService`; estrategia de notificación extensible
- **L**: Sustitución de Liskov — `R2dbcMatchRepository` debe ser 100% intercambiable con `MatchRepository` port
- **I**: Interfaces específicas por cliente — `MatchRepository` solo queries de match; `NotificationRepository` solo queries de notificación; no mezclar responsabilidades
- **D**: Depender de abstracciones, no implementaciones — `MatchService` recibe `MatchRepository` y `NotificationService` (interfaces), no implementaciones concretas

### Spring-Specific
- **Inyección por constructor** (no `@Autowired` en campos) — dependencias explícitas e inmutables en `MatchService`, `NotificationService`, ambos controllers
- `@ConfigurationProperties` para configuración de notificaciones (`notification.*`); nunca hardcodear
- **Eventos de Spring** (`ApplicationEventPublisher`) para publicar `MatchAcceptedEvent` que notifique a otros bounded contexts (ej. compartir datos de contacto, actualizar grafo en plan-06)
- `@Transactional` solo en services, con `readOnly = true` en consultas (`getHistory`, `getUnreadCount`); escritura en `expressInterest`, `accept`, `reject`, `markAsRead`
- `@RestControllerAdvice` + `@ExceptionHandler` para manejo centralizado de errores HTTP de interacciones
- `Bean Validation` (`@Valid`, `@NotNull`) en `MatchRequest`; no en `Match` entity

### Manejo de errores
- Lanzar excepciones específicas de dominio: `DuplicateMatchException`, `InvalidStateException` (transición no permitida), `UnauthorizedMatchActionException`, `MatchNotFoundException`
- `@ExceptionHandler` traduce excepciones de dominio a códigos HTTP: `DuplicateMatchException` → 409, `InvalidStateException` → 400, `UnauthorizedMatchActionException` → 403, `MatchNotFoundException` → 404
- Nunca exponer stack traces al cliente; loggear internamente con `log.error()`, retornar mensaje amigable en JSON `{"error": "Ya existe una interacción con este proyecto"}`
- Usar `Mono.error()` para flujos reactivos de error predecibles en `MatchService`
- Validar precondiciones al inicio del método con guard clauses: match existente, estado válido para la transición, actor autorizado

### Valores configurables
- Configuraciones de notificación, timeouts de match (si aplican) en `application.yml` bajo `match.*` y `notification.*`
- Usar `@ConfigurationProperties` para agrupar configuraciones de match (`MatchProperties`) y notificación (`NotificationProperties`)
- Perfiles (`application-dev.yml`, `application-prod.yml`) para separar configuraciones por entorno (ej. emails de notificación en prod)
- NUNCA hardcodear mensajes de notificación, URLs de redirect, timeouts o tipos de notificación en el código fuente

### Testing (si aplica)
- Tests nombrados `should_expectedBehavior_when_condition()` — `should_throwDuplicateMatchException_when_matchAlreadyExists()`, `should_throwInvalidStateException_when_acceptingNonInterestedMatch()`
- AAA: Arrange → Act → Assert, sin mezclar fases
- Un test = un concepto; mockear solo dependencias externas (`MatchRepository`, `NotificationService`), no `MatchService`
- `@SpringBootTest` solo para integración (POST /api/matches, POST /api/matches/{id}/accept); unitarios con Mockito puro para transiciones de estado
- Probar cada transición de estado (`INTERESADO → ACEPTADO`, `INTERESADO → RECHAZADO`) y transiciones inválidas de forma aislada

## Phase 1: Setup

- [ ] T001 Create V6__create_matches.sql Flyway migration (id, student_id FK, project_id FK, status, initiated_by, match_score, created_at, updated_at, UNIQUE(student_id, project_id))
- [ ] T002 Create V7__create_notifications.sql Flyway migration (id, user_id FK, match_id FK, type, message, is_read, created_at)
- [ ] T003 Create Match domain entity in domain/model/Match.java and Notification in domain/model/Notification.java
- [ ] T004 Create MatchRepository and NotificationRepository port interfaces in domain/port/
- [ ] T005 Create R2DBC adapter implementations in infrastructure/persistence/
- [ ] T006 Create MatchStatus enum (SUGERIDO, INTERESADO, ACEPTADO, RECHAZADO)
- [ ] T007 Create NotificationType enum (NUEVO_INTERES, MATCH_ACEPTADO, MATCH_RECHAZADO)

---

## Phase 2: Foundational — Notification Service

- [ ] T006 Implement NotificationService.createNotification(User recipient, Match match, NotificationType type): build message, persist Notification
- [ ] T007 Implement NotificationService.getUnreadCount(Long userId): count by userId where isRead=false
- [ ] T008 Implement NotificationService.markAsRead(Long notificationId): set isRead=true
- [ ] T008b Implement NotificationRestController.markAllAsRead(Long userId): set isRead=true for all notifications of user

---

## Phase 3: User Story 1 — Marcar interés (P1)

**Goal**: Estudiante o responsable marcan "me interesa" sobre una recomendación, creando un Match en estado INTERESADO con notificación a la otra parte.

**Independent Test**: POST /api/matches con studentId, projectId → Match INTERESADO creado, Notification generado para el receptor. POST duplicado → 409 Conflict.

### Tests for User Story 1

- [ ] T009 [US1] Unit test: MatchService.expressInterest(studentId, projectId, ESTUDIANTE) → Match INTERESADO, initiatedBy=ESTUDIANTE
- [ ] T010 [US1] Unit test: MatchService.expressInterest when match already exists → throws DuplicateMatchException
- [ ] T011 [US1] Unit test: Notification created for project owner when student expresses interest
- [ ] T012 [US1] Integration test: POST /api/matches → 201, Match in DB, Notification in DB

### Implementation for User Story 1

- [ ] T015 [US1] Implement MatchRestController.expressInterest (POST /api/matches): request body MatchRequest, returns Mono<ResponseEntity<MatchResponse>>
- [ ] T016 [US1] Implement MatchService.expressInterest: check uniqueness, create Match INTERESADO, determine recipient, call NotificationService.createNotification
- [ ] T017 [US1] Handle duplicate: return 409 Conflict with JSON error body "Ya existe una interacción con este proyecto"

---

## Phase 4: User Story 2 — Aceptar/Rechazar (P1)

**Goal**: Receptor acepta o rechaza un match INTERESADO. Match rechazado se excluye de futuras recomendaciones.

**Independent Test**: POST /api/matches/{id}/accept → status=ACEPTADO, notificación. POST /api/matches/{id}/reject → status=RECHAZADO, excluido de rankings.

### Tests for User Story 2

- [ ] T016 [US2] Unit test: MatchService.accept(matchId) → status=ACEPTADO, notification to initiator
- [ ] T017 [US2] Unit test: MatchService.reject(matchId) → status=RECHAZADO, notification to initiator
- [ ] T018 [US2] Unit test: MatchService.accept/reject on non-INTERESADO match → throws InvalidStateException
- [ ] T019 [US2] Unit test: MatchingService excludes RECHAZADO matches from rankings (integration with plan-04)

### Implementation for User Story 2

- [ ] T022 [US2] Implement MatchRestController.acceptMatch (POST /api/matches/{id}/accept): verify user is the recipient, call MatchService.accept, return Mono<ResponseEntity<MatchResponse>>
- [ ] T023 [US2] Implement MatchRestController.rejectMatch (POST /api/matches/{id}/reject): verify user is recipient, call MatchService.reject, return Mono<ResponseEntity<MatchResponse>>
- [ ] T024 [US2] Implement MatchService.accept: validate status=INTERESADO, update to ACEPTADO, notify initiator
- [ ] T025 [US2] Implement MatchService.reject: validate status=INTERESADO, update to RECHAZADO, notify initiator
- [ ] T026 [US2] Update MatchingController (plan-04) to filter out matches where status=RECHAZADO via MatchRepository

---

## Phase 5: User Story 3 — Historial de matches (P2)

**Goal**: Usuario ve lista de todos sus matches con estado actual, vía JSON.

**Independent Test**: GET /api/matches → JSON con lista de matches del usuario autenticado, agrupados/ordenados.

### Tests for User Story 3

- [ ] T025 [US3] Integration test: GET /api/matches as student → 200 with list of matches
- [ ] T026 [US3] Integration test: GET /api/matches as project owner → 200 with matches for owned projects

### Implementation for User Story 3

- [ ] T027 [US3] Implement MatchRestController.getMyMatches (GET /api/matches): for students → matches by studentId; for RESPONSABLE → matches by owned projectIds
- [ ] T028 [US3] Integrate unread notification count via NotificationService.getUnreadCount

---

## Dependencies & Execution Order

- **Phase 1-2**: Depend on plan-backend-02 (StudentProfile), plan-backend-03 (Project), plan-backend-01 (User)
- **US1 → US2**: US1 creates INTERESADO matches; US2 requires them
- **US3**: After US1 (needs matches to display)
- **Integration with plan-04**: Matching engine must query MatchRepository to exclude RECHAZADO pairs

---

## Notes

- Unicidad de Match garantizada por UNIQUE constraint en DB (via Flyway migration) y verificación en servicio
- Notificaciones in-app (entidad Notification persistida) y por correo electrónico.
- MatchScore se almacena en Match.matchScore como snapshot del puntaje al momento de la interacción
- Los matches en estado INTERESADO no expiran; quedan abiertos indefinidamente hasta que el receptor actúe.
- Match ACEPTADO es estado final e irreversible. No puede revertirse a RECHAZADO.
- Al aceptar un match, se comparten datos de contacto (nombre completo y correo electrónico de ambas partes).
- Clean code: meaningful names, small methods, SRP. No dead code, no commented-out code.
