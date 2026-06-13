# Implementation Plan: Backend — Datos para Red de Colaboración

**Date**: 2026-06-13
**Priority**: P3
**Spec**: `DOCS/specs/spec-backend-06-network-data.md`

## Summary

Implementar endpoint JSON que devuelve los datos estructurados del grafo de colaboración (nodos y aristas) basados en matches ACEPTADOS, más agregaciones por facultad. Consumido por el frontend React para renderizar la visualización de red.

## Technical Context

**Language/Version**: Java 21
**Primary Dependencies**: Spring Boot 3.x, Spring Data JPA, Lombok
**Storage**: PostgreSQL (consultas JPQL/nativas sobre entidades existentes Match, StudentProfile, Project)
**Testing**: JUnit 5, Mockito, Spring Boot Test, H2
**Target Platform**: Server (Spring Boot embedded Tomcat)
**Project Type**: web (monolith with layers)

## Project Structure

```text
src/main/java/com/unimag/match/
├── controller/
│   └── NetworkApiController.java     # @RestController: GET /api/network/graph, GET /api/network/faculty-stats
├── service/
│   └── NetworkService.java           # buildGraphData, getFacultyAggregations
├── dto/
│   ├── GraphDataDto.java             # { nodes: [...], edges: [...] }
│   ├── GraphNodeDto.java             # { id, label, type, faculty? }
│   ├── GraphEdgeDto.java             # { source, target, weight, type }
│   └── FacultyStatsDto.java          # { faculty, studentCount, projectCount, topSkills }
├── repository/
│   ├── MatchRepository.java          # (exists from plan-05, add custom queries)
│   └── StudentProfileRepository.java # (exists from plan-02, add aggregation queries)
```

## Phase 1: Setup

- [ ] T001 Create DTOs: GraphDataDto, GraphNodeDto (id, label, type: ESTUDIANTE|FACULTAD, faculty), GraphEdgeDto (source, target, weight, type: MATCH|FACULTY_CONNECTION), FacultyStatsDto
- [ ] T002 Add custom query methods to MatchRepository: countMatchesByFacultyPair, findMatchesByFaculty

---

## Phase 2: User Story 1 — Generar datos del grafo (P3)

**Goal**: Endpoint JSON que devuelve nodos (estudiantes, facultades) y aristas (matches entre estudiantes, conexiones entre facultades con peso).

**Independent Test**: Con datos de prueba de matches ACEPTADOS → JSON con nodos y aristas correctos, peso inter-facultad proporcional al número de matches.

### Tests for User Story 2

- [ ] T003 [US1] Unit test: NetworkService.buildGraphData → nodes include all students with accepted matches and their faculties
- [ ] T004 [US1] Unit test: NetworkService.buildGraphData → edges include student-student match connections
- [ ] T005 [US1] Unit test: NetworkService.buildGraphData → inter-faculty edge weight equals number of matches between those faculties
- [ ] T006 [US1] Integration test: GET /api/network/graph → 200 JSON with expected structure
- [ ] T007 [US1] Integration test: GET /api/network/graph with no accepted matches → empty nodes/edges or message

### Implementation for User Story 1

- [ ] T008 [US1] Implement NetworkService.buildGraphData():
  - Query all accepted matches with student profiles (JPQL join)
  - Build student nodes: id=studentId, label=fullName, type=ESTUDIANTE, faculty
  - Derive faculty nodes: distinct faculties from student nodes, type=FACULTAD
  - Build match edges: source=studentA, target=studentB (or student-project), type=MATCH
  - Build faculty edges: aggregate count of matches between faculty pairs, weight=count, type=FACULTY_CONNECTION
  - Map weight to 3 intensity levels: BAJA (1-2), MEDIA (3-5), ALTA (6+)
- [ ] T009 [US1] Implement NetworkApiController.getGraph (GET /api/network/graph): optional @RequestParam faculty filter
- [ ] T010 [US1] Implement faculty filter: when faculty param present, return only nodes/edges related to that faculty

---

## Phase 3: User Story 2 — Agregaciones por facultad (P3)

**Goal**: Endpoint JSON con estadísticas por facultad: conteo de estudiantes, proyectos, top habilidades.

**Independent Test**: GET /api/network/faculty-stats → JSON con arrays de FacultyStatsDto.

### Tests for User Story 2

- [ ] T011 [US2] Unit test: NetworkService.getFacultyAggregations → correct student counts per faculty
- [ ] T012 [US2] Unit test: NetworkService.getFacultyAggregations → top skills per faculty ordered by frequency
- [ ] T013 [US2] Integration test: GET /api/network/faculty-stats → 200 JSON with expected structure

### Implementation for User Story 2

- [ ] T014 [US2] Implement NetworkService.getFacultyAggregations():
  - JPQL query: SELECT sp.faculty, COUNT(sp), ... FROM StudentProfile sp GROUP BY sp.faculty
  - Top skills: aggregate skills per faculty using @ManyToMany join, count frequency, limit to top 5
  - Project count per faculty: from Project.relatedFaculty
- [ ] T015 [US2] Implement NetworkApiController.getFacultyStats (GET /api/network/faculty-stats)

---

## Dependencies & Execution Order

- **Phase 1-2**: Depend on plan-backend-05 (Match entity with ACEPTADO status), plan-backend-02 (StudentProfile), plan-backend-03 (Project)
- **US2**: After US1 (can be parallel, but uses same service)
- **This module is P3** — implement only after all P1 modules are complete

---

## Notes

- El grafo se calcula on-the-fly (no se persiste). Para muestra piloto de 120 estudiantes el rendimiento es suficiente.
- Los nodos de estudiantes se incluyen solo si tienen al menos un match ACEPTADO (sin nodos aislados).
- Peso intra-facultad (matches entre estudiantes de la misma facultad): se incluye como arista con source=target=facultad, weight=count.
- Frontend React recibe JSON y renderiza usando biblioteca de visualización de grafos (a definir en fase frontend).
