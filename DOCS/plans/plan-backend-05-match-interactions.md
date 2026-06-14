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
- Notificaciones solo in-app (entidad persistida). [NEEDS CLARIFICATION: ¿email? No implementado en MVP]
- MatchScore se almacena en Match.matchScore como snapshot del puntaje al momento de la interacción
- [NEEDS CLARIFICATION]: ¿Los matches INTERESADO expiran? No implementado en MVP; quedan abiertos indefinidamente
- [NEEDS CLARIFICATION]: ¿Match ACEPTADO puede revertirse a RECHAZADO? No especificado; asumimos que ACEPTADO es estado final
