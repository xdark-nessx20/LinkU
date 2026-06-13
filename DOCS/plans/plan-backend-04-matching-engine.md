# Implementation Plan: Backend — Motor de Matching y Recomendación

**Date**: 2026-06-13
**Spec**: `DOCS/specs/spec-backend-04-matching-engine.md`

## Summary

Implementar el motor de puntuación ponderada (+30/+15/+20/+20/+15) que calcula compatibilidad entre cada estudiante y proyecto activo. Generar rankings bidireccionales (estudiante→proyectos, proyecto→estudiantes) con factores explicativos, expuestos como endpoints JSON internos para el frontend React.

## Technical Context

**Language/Version**: Java 21
**Primary Dependencies**: Spring Boot 3.x, Spring Data JPA, Lombok
**Storage**: PostgreSQL (entidad MatchScore con score_factors como JSONB)
**Testing**: JUnit 5, Mockito, Spring Boot Test, H2
**Target Platform**: Server (Spring Boot embedded Tomcat)
**Project Type**: web (monolith with layers)

## Project Structure

```text
src/main/java/com/unimag/match/
├── config/
│   └── MatchingConfig.java           # @ConfigurationProperties para pesos (matching.weights.skill=30, etc.)
├── controller/
│   └── MatchingApiController.java    # @RestController: GET /api/recommendations/{userId}, GET /api/projects/{id}/recommendations
├── model/
│   ├── StudentProfile.java
│   ├── Project.java
│   ├── Match.java                    # (from plan-05)
│   └── MatchScore.java              # @Entity: studentId, projectId, totalScore, skillScore, programScore, interestScore, experienceScore, availabilityScore, scoreFactors (JSONB), calculatedAt
├── repository/
│   └── MatchScoreRepository.java     # findByStudentId, findByProjectId, deleteByStudentId/ProjectId
├── service/
│   ├── MatchingService.java          # calculateAll, calculateForStudent, calculateForProject, scoring logic
│   └── ScoringWeights.java           # @ConfigurationProperties class
└── dto/
    ├── RecommendationDto.java        # JSON: project/student info, totalScore, factors[], explanation
    └── ScoreFactorsDto.java          # Desglose JSONB mapeado

src/main/resources/
└── application.properties            # matching.weights.skill=30, etc.
```

## Phase 1: Setup

- [ ] T001 Create MatchScore entity with JSONB column for scoreFactors (PostgreSQL jsonb type via Hibernate @JdbcTypeCode(SqlTypes.JSON) or @Column(columnDefinition = "jsonb"))
- [ ] T002 Create MatchScoreRepository
- [ ] T003 Create ScoringWeights as @ConfigurationProperties (prefix = "matching.weights")
- [ ] T004 Add matching.weights.* to application.properties with defaults (+30/+15/+20/+20/+15)
- [ ] T005 Create RecommendationDto and ScoreFactorsDto

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

**Goal**: Endpoint JSON que devuelve proyectos recomendados para un estudiante, ordenados por puntaje, con explicación.

**Independent Test**: GET /api/recommendations/{userId} → JSON array ordenado por totalScore desc, cada entrada con totalScore y factors[].

### Tests for User Story 2

- [ ] T018 [US2] Integration test: GET /api/recommendations/{userId} → 200, JSON with projects sorted desc by score
- [ ] T019 [US2] Integration test: recommendation excludes rejected matches (loads Match from DB)
- [ ] T020 [US2] Integration test: recommendation includes score breakdown in response

### Implementation for User Story 2

- [ ] T021 [US2] Implement MatchingApiController.getRecommendationsForStudent (GET /api/recommendations/{userId})
- [ ] T022 [US2] Implement recommendation filtering: exclude inactive projects, exclude rejected matches, exclude if student profile is null/incomplete
- [ ] T023 [US2] Build RecommendationDto with project info, totalScore, and scoreFactors explanation
- [ ] T024 [US2] Handle empty profile case: if student has no skills/interests → return empty array with message "Completa tu perfil..."

---

## Phase 5: User Story 3 — Rankings para proyecto (P1)

**Goal**: Endpoint JSON que devuelve estudiantes recomendados para un proyecto.

**Independent Test**: GET /api/projects/{id}/recommendations → JSON array de estudiantes ordenados por score.

### Tests for User Story 3

- [ ] T025 [US3] Integration test: GET /api/projects/{id}/recommendations → students sorted by score desc
- [ ] T026 [US3] Integration test: project with no matches above threshold → empty array with suggestion message

### Implementation for User Story 3

- [ ] T027 [US3] Implement MatchingApiController.getRecommendationsForProject (GET /api/projects/{projectId}/recommendations)
- [ ] T028 [US3] Apply same filtering logic as US2 (exclude inactive profiles, rejected matches)
- [ ] T029 [US3] Handle no-matches case: return empty with "No se encontraron coincidencias fuertes" message

---

## Phase 6: User Story 4 — Recalcular ante cambios (P2)

**Goal**: Cuando un perfil o proyecto cambia, las recomendaciones se recalculan bajo demanda.

### Implementation for User Story 4

- [ ] T030 [US4] Implement MatchScoreRepository.deleteByStudentId and deleteByProjectId (called before recalculation)
- [ ] T031 [US4] Trigger recalculation in ProfileService.update → delete existing MatchScores for that student, recalculate
- [ ] T032 [US4] Trigger recalculation in ProjectService.update → delete existing MatchScores for that project, recalculate

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
- [NEEDS CLARIFICATION]: Experiencia previa "relacionada" usa matching de palabras clave simple (contains/JPQL LIKE) en MVP. Mejorar en fase post-piloto.
- MatchScore se persiste para auditoría y rendimiento. Se borra y recalcula ante cambios relevantes para mantener consistencia.
