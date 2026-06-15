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

## Clean Code Guidelines

### Naming & Style
- **Clases/Interfaces**: `PascalCase` — `MatchingService`, `MatchingScorer`, `MatchScoreRepository`, `MatchingRestController`, `MatchingConfig`
- **Métodos**: `camelCase` — `calculateMatch()`, `calculateForStudent()`, `calculateForProject()`, `getRecommendations()`, `normalizeScore()`
- **Constantes**: `UPPER_SNAKE_CASE` — `MIN_VISIBILITY_THRESHOLD`, `SCORE_SCALE_MAX`, `DEFAULT_WEIGHT_SKILL`
- **Paquetes**: `lowercase` sin guiones — `com.unimag.match.domain.service`, `com.unimag.match.application.dto`
- **DTOs**: Sufijo explícito — `RecommendationResponse` (respuesta JSON con score y factores), `ScoreBreakdownDto` (desglose de factores)

### Single Responsibility
- Cada clase debe tener UNA razón para cambiar
- **Controllers**: Solo HTTP — `MatchingRestController` recibe peticiones de rankings, delega a `MatchingService`, retorna `Mono<ResponseEntity<List<RecommendationResponse>>>`
- **Services**: Solo lógica de negocio — `MatchingService` orquesta el cálculo de scores, normalización y filtrado; `MatchingScorer` (domain service) encapsula el algoritmo puro de puntuación por factor; nunca acceden a HTTP ni a DB directamente
- **Repositories**: Solo acceso a datos — `MatchScoreRepository` persiste y consulta scores; no contiene lógica de cálculo
- **Domain/Entities**: Solo datos y validaciones básicas — `MatchScore` encapsula score total, factores (JSONB) y relaciones a student/project
- Si una clase supera ~300 líneas, extraer responsabilidades (ej. `ScoreNormalizer`, `TieBreakResolver` separados de `MatchingService`)
- Separar paquetes por dominio: `matching` agrupa service, dto, config; `MatchingScorer` en `domain/service` por ser lógica de dominio pura

### Métodos limpios
- Máximo ~30 líneas por método; extraer bloques lógicos a métodos privados con nombre descriptivo (ej. `calculateSkillScore`, `calculateInterestScore`, `applyTieBreakRules`)
- **Guard clauses al inicio**: `if (studentProfile == null) return Mono.just(Collections.emptyList())` — retornar o lanzar temprano
- Un solo nivel de abstracción por método: `calculateMatch` delega a métodos por factor (skill, program, interest, experience, availability); no mezcla el cálculo de cada factor con la suma total
- Evitar más de 3 niveles de indentación; usar `Stream` para iterar pares estudiante-proyecto y `Optional` para factores opcionales
- Métodos de consulta (`findByStudentId`, `findByProjectId`) deben ser side-effect-free; métodos de comando (`calculateAll`, `deleteByStudentId`) deben documentar el efecto

### Principios SOLID
- **S**: Single Responsibility (ver arriba)
- **O**: Abierto a extensión — `MatchingScorer` como interfaz de dominio permite nuevas estrategias de scoring sin modificar `MatchingService`; pesos configurables permiten ajustar sin recompilar
- **L**: Sustitución de Liskov — cualquier implementación de `MatchingScorer` debe ser intercambiable sin afectar `MatchingService`
- **I**: Interfaces específicas por cliente — `MatchScoreRepository` solo queries de score; `MatchingScorer` solo interfaz de cálculo
- **D**: Depender de abstracciones, no implementaciones — `MatchingService` depende de `MatchingScorer` (interfaz) y `MatchScoreRepository` (interfaz), no de implementaciones concretas

### Spring-Specific
- **Inyección por constructor** (no `@Autowired` en campos) — dependencias explícitas e inmutables en `MatchingService`, `MatchingRestController`, `MatchingScorer`
- `@ConfigurationProperties(prefix = "matching.weights")` para pesos de scoring (`skill`, `program`, `interest`, `experience`, `availability`); ajustables sin recompilar
- `@ConfigurationProperties(prefix = "matching")` para umbral de visibilidad (`min-visibility-threshold=50`)
- **Eventos de Spring** (`ApplicationEventPublisher`) para escuchar `ProfileUpdatedEvent` y `ProjectUpdatedEvent` que gatillen recálculo bajo demanda
- `@Transactional` solo en services, con `readOnly = true` en consultas de rankings; escritura en `saveMatchScore`, `deleteByStudentId`
- `@RestControllerAdvice` + `@ExceptionHandler` para manejo centralizado de errores HTTP del motor de matching

### Manejo de errores
- Lanzar excepciones específicas de dominio: `InsufficientDataException` (perfil sin skills/proyecto sin requerimientos), `CalculationException` (error en cómputo de score)
- `@ExceptionHandler` traduce excepciones de dominio a códigos HTTP: `InsufficientDataException` → 200 con lista vacía (no es error), `CalculationException` → 500
- Nunca exponer stack traces al cliente; loggear internamente con `log.error()`, retornar mensaje amigable en JSON
- Usar `Mono.error()` o `Mono.just(emptyList())` según corresponda para flujos reactivos predecibles
- Validar precondiciones al inicio del método: perfiles y proyectos existentes, datos suficientes para calcular score

### Valores configurables
- Pesos de scoring: `matching.weights.skill=30`, `matching.weights.program=15`, `matching.weights.interest=20`, `matching.weights.experience=20`, `matching.weights.availability=15`
- Umbral mínimo de visibilidad: `matching.min-visibility-threshold=50` (50%)
- Usar `@ConfigurationProperties` para agrupar todas las configuraciones de matching (`MatchingProperties` con subgrupo `WeightsProperties`)
- Perfiles (`application-dev.yml`, `application-prod.yml`) para ajustar pesos y umbrales por entorno sin recompilar
- NUNCA hardcodear pesos, umbrales, o fórmulas de normalización en el código fuente

### Testing (si aplica)
- Tests nombrados `should_expectedBehavior_when_condition()` — `should_returnSkillScore30_when_skillMatches()`, `should_returnZero_when_noMatches()`
- AAA: Arrange → Act → Assert, sin mezclar fases
- Un test = un concepto; mockear solo dependencias externas (`MatchScoreRepository`, repos de perfil/proyecto), no `MatchingScorer`
- `@SpringBootTest` solo para integración (GET /api/recommendations/student/{id}); unitarios con Mockito puro para cada factor de scoring
- Probar cada factor de scoring de forma aislada con datos de entrada controlados

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
  - Skill matching: count overlapping skills between student.skills and project.requiredSkills → skillScore = count × weight.skill (sin tope, acumulativo por cada habilidad)
  - Program matching: if student.program matches or relates to project.relatedFaculty → weight.program (15)
  - Interest matching: if any student.interest overlaps with project.mainObjective/topic → weight.interest (20)
  - Experience matching: if student.priorExperience relates to project description by keyword matching and thematic category → weight.experience (20)
  - Availability: if student.availability matches project.expectedAvailability → weight.availability (15)
  - totalScore = skillScore + programScore + interestScore + experienceScore + availabilityScore (puede exceder 100)
  - normalizedScore = totalScore relativo al máximo del conjunto, escalado a 0-100 (ej. si el máximo es 125, 125→100%, 62.5→50%)
  - scoreFactors: JSON construido con breakdown de cada factor
- [ ] T007 Implement score normalization: normalizedScore = (totalScore / maxTotalScoreInSet) × 100 para porcentaje relativo. Solo se incluyen en rankings recomendaciones con normalizedScore ≥ 50%.
- [ ] T008 Implement tie-breaking: when normalizedScore equals → order by studentProfile.updatedAt desc, then studentProfile.fullName asc

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
- SkillScore: +30 por cada habilidad coincidente, sin tope. La normalización se hace de forma relativa al máximo puntaje del conjunto.
- Score normalization: porcentaje relativo al máximo del conjunto (puntaje bruto puede exceder 100). Umbral mínimo de visualización: ≥50%.
- Experiencia previa "relacionada" usa matching de palabras clave y categoría temática en el MVP. Mejorar en fase post-piloto.
- MatchScore se persiste para auditoría y rendimiento. Se borra y recalcula bajo demanda ante cambios relevantes.
- Cálculo bajo demanda por usuario (no precalculado periódicamente). Se ejecuta al solicitar rankings.
- Rankings via REST endpoints GET /api/recommendations/student/{id} y GET /api/recommendations/project/{id}. Respuestas JSON.
- Criterios de desempate: (1) fecha de actualización de perfil más reciente, (2) orden alfabético por nombre.
- Clean code: meaningful names, small methods, SRP. No dead code, no commented-out code.
