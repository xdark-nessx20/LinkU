# Implementation Plan: Backend — Motor de Matching y Recomendación

**Date**: 2026-06-13
**Spec**: `DOCS/specs/spec-backend-04-matching-engine.md`

## Summary

Implementar el motor de puntuación ponderada (+30/+15/+20/+20/+15) que calcula compatibilidad entre cada estudiante y proyecto activo. Generar rankings bidireccionales (estudiante→proyectos, proyecto→estudiantes) con factores explicativos, expuestos como endpoints JSON internos para el frontend React.

## Technical Context

**Language/Version**: Java 21
**Primary Dependencies**: Spring Boot 3.x, Spring Data R2DBC, Lombok
**Storage**: PostgreSQL (entidad MatchScore con score_factors como JSONB)
**Testing**: JUnit 5, Mockito, Spring Boot Test, H2
**Target Platform**: Server (Spring Boot WebFlux on Netty)
**Project Type**: web (hexagonal with ports & adapters)

## Project Structure

```text
src/main/java/com/unimag/match/
├── domain/
│   ├── model/
│   │   ├── StudentProfile.java
│   │   ├── Project.java
│   │   ├── Match.java                # (from plan-05)
│   │   └── MatchScore.java           # Domain entity
│   ├── service/
│   │   └── MatchingScorer.java       # Domain interface: calculateMatch(profile, project)
│   └── port/
│       └── MatchScoreRepository.java # Port interface
├── application/
│   ├── service/
│   │   └── MatchingService.java      # Use case: calculateAll, calculateForStudent, calculateForProject
│   └── dto/
│       └── RecommendationResponse.java  # JSON response DTO
└── infrastructure/
    ├── config/
    │   └── MatchingConfig.java        # @ConfigurationProperties para pesos
    ├── persistence/
    │   ├── R2dbcMatchScoreRepository.java
    │   └── R2dbcJsonbType.java       # JSONB handler for R2DBC
    └── web/
        └── MatchingRestController.java  # @RestController: GET /api/recommendations/*

src/main/resources/
├── db/migration/
│   └── V5__create_match_scores.sql   # Flyway migration
└── application.properties            # matching.weights.skill=30, etc.
```

## Phase 1: Setup

- [ ] T001 Create V5__create_match_scores.sql Flyway migration with JSONB column for scoreFactors
- [ ] T002 Create MatchScore domain entity in domain/model/MatchScore.java
- [ ] T003 Create MatchScoreRepository port interface in domain/port/
- [ ] T004 Create R2dbcMatchScoreRepository adapter in infrastructure/persistence/ (with JSONB handling)
- [ ] T005 Create ScoringWeights as @ConfigurationProperties (prefix = "matching.weights")
- [ ] T006 Add matching.weights.* to application.properties with defaults (+30/+15/+20/+20/+15)
- [ ] T007 Create RecommendationResponse DTO in application/dto/RecommendationResponse.java

---

## Phase 2: Foundational — Scoring Logic

- [ ] T006 Implement MatchingService.calculateMatch(StudentProfile, Project): MatchScore
  - Skill matching: count overlapping skills between student.skills and project.requiredSkills → skillScore = count × weight.skill, capped at 60
  - Program matching: if student.program matches or relates to project.relatedFaculty → weight.program (15)
  - Interest matching: if any student.interest overlaps with project.mainObjective/topic → weight.interest (20)
  - Experience matching: if student.priorExperience relates to project description → weight.experience (20) [NEEDS CLARIFICATION: matching por palabras clave básico por ahora]
  - Availability: if student.availability matches project.expectedAvailability → weight.availability (15)
  - totalScore = skillScore + programScore + interestScore + experienceScore + availabilityScore, normalized to 0-100
  - scoreFactors: JSON construido con breakdown de cada factor
- [ ] T007 Implement score normalization: min(totalScore, 100) for capping at 100%
- [ ] T008 Implement tie-breaking: when totalScore equals → order by studentProfile.updatedAt desc, then studentProfile.fullName asc

---

## Phase 3: User Story 1 — Cálculo de coincidencias (P1)

**Goal**: Motor calcula puntaje para cada par (estudiante, proyecto activo).

**Independent Test**: Con 3 estudiantes y 2 proyectos de prueba → se calculan 6 MatchScore objetos con puntajes correctos y factores desglosados.

### Tests for User Story 1

- [ ] T009 [US1] Unit test: MatchingService.calculateMatch — student with matching skill "Python" → skillScore=30
- [ ] T010 [US1] Unit test: MatchingService.calculateMatch — student with matching program → programScore=15
- [ ] T011 [US1] Unit test: MatchingService.calculateMatch — student with matching interest → interestScore=20
- [ ] T012 [US1] Unit test: MatchingService.calculateMatch — student with no matches → totalScore=0
- [ ] T013 [US1] Unit test: MatchingService.calculateMatch — score capped at 100 when exceeding
- [ ] T014 [US1] Integration test: calculateAll with sample data → correct number of MatchScore rows

### Implementation for User Story 1

- [ ] T015 [US1] Implement MatchingService.calculateForStudent(Long userId): loads projects, calculates scores, persists MatchScore rows, returns sorted list
- [ ] T016 [US1] Implement MatchingService.calculateForProject(Long projectId): loads students, calculates scores, persists
- [ ] T017 [US1] Implement MatchingService.calculateAll(): batch calculates all student×project pairs (used for initial data load or admin trigger)

---

## Phase 4: User Story 2 — Rankings para estudiante (P1)

**Goal**: REST endpoint que retorna proyectos recomendados para un estudiante, ordenados por puntaje, con explicación de factores.

**Independent Test**: GET /api/recommendations/student/{userId} → JSON con proyectos ordenados por totalScore desc, cada uno con totalScore y factores desglosados.

### Tests for User Story 2

- [ ] T020 [US2] Integration test: GET /api/recommendations/student/{userId} → 200, JSON with projects sorted desc by score
- [ ] T021 [US2] Integration test: recommendation excludes rejected matches
- [ ] T022 [US2] Integration test: recommendation includes score breakdown in JSON response

### Implementation for User Story 2

- [ ] T023 [US2] Implement MatchingRestController.getRecommendationsForStudent (GET /api/recommendations/student/{userId}): retorna Mono<ResponseEntity<List<RecommendationResponse>>>
- [ ] T024 [US2] Implement recommendation filtering: exclude inactive projects, exclude rejected matches, exclude if student profile is null/incomplete
- [ ] T025 [US2] Map MatchScore entities to RecommendationResponse with project info, totalScore, and scoreFactors
- [ ] T026 [US2] Handle empty profile case: if student has no skills/interests → return empty list with 200 OK

---

## Phase 5: User Story 3 — Rankings para proyecto (P1)

**Goal**: REST endpoint que retorna estudiantes recomendados para un proyecto.

**Independent Test**: GET /api/recommendations/project/{projectId} → JSON con estudiantes ordenados por score.

### Tests for User Story 3

- [ ] T027 [US3] Integration test: GET /api/recommendations/project/{projectId} → 200, JSON with students sorted by score desc
- [ ] T028 [US3] Integration test: project with no matches above threshold → returns empty list with 200 OK

### Implementation for User Story 3

- [ ] T029 [US3] Implement MatchingRestController.getRecommendationsForProject (GET /api/recommendations/project/{projectId})
- [ ] T030 [US3] Apply same filtering logic as US2
- [ ] T031 [US3] Handle no-matches case: return empty list with 200 OK

---

## Phase 6: User Story 4 — Recalcular ante cambios (P2)

**Goal**: Cuando un perfil o proyecto cambia, las recomendaciones se recalculan bajo demanda.

### Implementation for User Story 4

- [ ] T032 [US4] Implement MatchScoreRepository.deleteByStudentId and deleteByProjectId (called before recalculation)
- [ ] T033 [US4] Trigger recalculation in ProfileService.update → delete existing MatchScores for that student, recalculate
- [ ] T034 [US4] Trigger recalculation in ProjectService.update → delete existing MatchScores for that project, recalculate

---

## Dependencies & Execution Order

- **Phase 1-2**: Depend on plan-backend-02 (StudentProfile, SkillCategory) and plan-backend-03 (Project)
- **US1 → US2/US3**: US1 scoring logic needed for rankings
- **US2/US3**: Parallel
- **US4**: After US2/US3 (needs recalculation trigger in profile/project services)

---

## Notes

- Pesos configurados en application.properties bajo matching.weights.* para fácil calibración
- SkillScore: +30 por cada habilidad coincidente, pero cap total de skillScore a 60 (máximo 2 habilidades cuentan). Esto evita que un estudiante con muchas habilidades sobrepase 100 siempre.
- Score normalization: totalScore = min(rawScore, 100)
- [NEEDS CLARIFICATION]: Experiencia previa "relacionada" usa matching de palabras clave simple en MVP. Mejorar en fase post-piloto.
- MatchScore se persiste para auditoría y rendimiento. Se borra y recalcula ante cambios relevantes.
- Rankings via REST endpoints GET /api/recommendations/student/{id} y GET /api/recommendations/project/{id}. Respuestas JSON.
