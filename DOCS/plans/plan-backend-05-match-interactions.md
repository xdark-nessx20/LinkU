# Implementation Plan: Backend — Gestión de Interacciones (Match)

**Date**: 2026-06-13
**Spec**: `DOCS/specs/spec-backend-05-match-interactions.md`

## Summary

Implementar el ciclo de vida del match: marcar "me interesa", aceptar/rechazar, notificaciones in-app persistidas, historial de matches por usuario. Exponer endpoints JSON para las acciones de interacción.

## Technical Context

**Language/Version**: Java 21
**Primary Dependencies**: Spring Boot 3.x, Spring Data JPA, Lombok, Thymeleaf
**Storage**: PostgreSQL (entidades Match, Notification)
**Testing**: JUnit 5, Mockito, Spring Boot Test, H2
**Target Platform**: Server (Spring Boot embedded Tomcat)
**Project Type**: web (monolith with layers)

## Project Structure

```text
src/main/java/com/unimag/match/
├── controller/
│   ├── MatchController.java          # GET HTML pages: match history
│   └── MatchApiController.java       # @RestController: POST /api/matches (express interest), POST /api/matches/{id}/accept, POST /api/matches/{id}/reject
├── model/
│   ├── Match.java                    # @Entity: studentId (FK), projectId (FK), status (enum), initiatedBy (enum), matchScore, createdAt, updatedAt. UniqueConstraint(studentId, projectId)
│   └── Notification.java            # @Entity: userId (FK), matchId (FK), type (enum), message, isRead, createdAt
├── repository/
│   ├── MatchRepository.java          # findByStudentId, findByProjectId, findByStudentIdAndProjectId
│   └── NotificationRepository.java   # findByUserIdOrderByCreatedAtDesc, countByUserIdAndIsReadFalse
├── service/
│   ├── MatchService.java             # expressInterest, accept, reject, getHistory
│   └── NotificationService.java      # createNotification, markAsRead, getUnreadCount
└── dto/
    ├── MatchDto.java                 # JSON response for match data
    └── NotificationDto.java

src/main/resources/
└── templates/
    └── match/
        ├── list.html                 # Historial de matches
        └── detail.html               # Detalle de un match
```

## Phase 1: Setup

- [ ] T001 Create Match entity: id, studentId (@ManyToOne StudentProfile), projectId (@ManyToOne Project), status (@Enumerated STRING: SUGERIDO, INTERESADO, ACEPTADO, RECHAZADO), initiatedBy (@Enumerated STRING: ESTUDIANTE, RESPONSABLE), matchScore (Integer 0-100), createdAt, updatedAt. @Table(uniqueConstraints = @UniqueConstraint(columnNames = {"student_id", "project_id"}))
- [ ] T002 Create Notification entity: id, userId (@ManyToOne User), matchId (@ManyToOne Match), type (@Enumerated STRING: NUEVO_INTERES, MATCH_ACEPTADO, MATCH_RECHAZADO), message (String), isRead (boolean default false), createdAt
- [ ] T003 Create MatchRepository and NotificationRepository
- [ ] T004 Create MatchStatus enum (SUGERIDO, INTERESADO, ACEPTADO, RECHAZADO)
- [ ] T005 Create NotificationType enum (NUEVO_INTERES, MATCH_ACEPTADO, MATCH_RECHAZADO)

---

## Phase 2: Foundational — Notification Service

- [ ] T006 Implement NotificationService.createNotification(User recipient, Match match, NotificationType type): build message, persist Notification
- [ ] T007 Implement NotificationService.getUnreadCount(Long userId): count by userId where isRead=false
- [ ] T008 Implement NotificationService.markAsRead(Long notificationId): set isRead=true

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

- [ ] T013 [US1] Implement MatchApiController.expressInterest (POST /api/matches): body {studentId, projectId}
- [ ] T014 [US1] Implement MatchService.expressInterest: check uniqueness, create Match INTERESADO, determine recipient (if initiator=ESTUDIANTE → project.owner, if initiator=RESPONSABLE → student.user), call NotificationService.createNotification
- [ ] T015 [US1] Handle duplicate: return 409 Conflict with message "Ya existe una interacción con este proyecto"

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

- [ ] T020 [US2] Implement MatchApiController.acceptMatch (POST /api/matches/{id}/accept): verify user is the recipient (not initiator), call MatchService.accept
- [ ] T021 [US2] Implement MatchApiController.rejectMatch (POST /api/matches/{id}/reject): verify user is recipient, call MatchService.reject
- [ ] T022 [US2] Implement MatchService.accept: validate status=INTERESADO, update to ACEPTADO, notify initiator
- [ ] T023 [US2] Implement MatchService.reject: validate status=INTERESADO, update to RECHAZADO, notify initiator
- [ ] T024 [US2] Update MatchingApiController (plan-04) to filter out matches where status=RECHAZADO via MatchRepository

---

## Phase 5: User Story 3 — Historial de matches (P2)

**Goal**: Usuario ve lista de todos sus matches con estado actual, vía HTML.

**Independent Test**: GET /matches → HTML con lista de matches del usuario autenticado, agrupados/ordenados.

### Tests for User Story 3

- [ ] T025 [US3] Integration test: GET /matches as student → 200 with list of matches
- [ ] T026 [US3] Integration test: GET /matches as project owner → 200 with matches for owned projects

### Implementation for User Story 3

- [ ] T027 [US3] Implement MatchController.listMyMatches (GET /matches): for students → matches by studentId; for RESPONSABLE → matches by owned projectIds
- [ ] T028 [US3] Create templates/match/list.html: table with project/student name, status badge, date, actions
- [ ] T029 [US3] Integrate unread notification count in navbar (via NotificationService.getUnreadCount)

---

## Dependencies & Execution Order

- **Phase 1-2**: Depend on plan-backend-02 (StudentProfile), plan-backend-03 (Project), plan-backend-01 (User)
- **US1 → US2**: US1 creates INTERESADO matches; US2 requires them
- **US3**: After US1 (needs matches to display)
- **Integration with plan-04**: Matching engine must query MatchRepository to exclude RECHAZADO pairs

---

## Notes

- Unicidad de Match garantizada por @UniqueConstraint en DB y verificación en servicio
- Notificaciones solo in-app (entidad persistida). [NEEDS CLARIFICATION: ¿email? No implementado en MVP]
- MatchScore se almacena en Match.matchScore como snapshot del puntaje al momento de la interacción
- [NEEDS CLARIFICATION]: ¿Los matches INTERESADO expiran? No implementado en MVP; quedan abiertos indefinidamente
- [NEEDS CLARIFICATION]: ¿Match ACEPTADO puede revertirse a RECHAZADO? No especificado; asumimos que ACEPTADO es estado final
