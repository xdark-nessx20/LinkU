# Implementation Plan: Backend — Datos para Red de Colaboración

**Date**: 2026-06-13
**Priority**: P3
**Spec**: `DOCS/specs/spec-backend-06-network-data.md`

## Summary

Implementar endpoint JSON que devuelve los datos estructurados del grafo de colaboración (nodos y aristas) basados en matches ACEPTADOS, más agregaciones por facultad. Consumido por el frontend React para renderizar la visualización de red.

## Technical Context

**Language/Version**: Java 21
**Primary Dependencies**: Spring Boot 3.x, Spring Data R2DBC, Lombok
**Storage**: PostgreSQL (consultas sobre entidades existentes Match, StudentProfile, Project)
**Testing**: JUnit 5, Mockito, Spring Boot Test, H2
**Target Platform**: Server (Spring Boot WebFlux on Netty)
**Project Type**: web (hexagonal with ports & adapters)

## Project Structure

```text
src/main/java/com/unimag/match/
├── domain/
│   ├── service/
│   │   └── GraphDataBuilder.java      # Domain interface: build graph from accepted matches
│   └── port/
│       ├── MatchRepository.java        # (exists from plan-05, add custom queries)
│       └── StudentProfileRepository.java # (exists from plan-02, add aggregation queries)
├── application/
│   ├── service/
│   │   └── NetworkService.java        # Use case: buildGraphData, getFacultyAggregations
│   └── dto/
│       ├── GraphDataDto.java           # { nodes: [...], edges: [...] }
│       ├── GraphNodeDto.java           # { id, label, type, faculty? }
│       ├── GraphEdgeDto.java           # { source, target, weight, type }
│       └── FacultyStatsDto.java        # { faculty, studentCount, projectCount, topSkills }
└── infrastructure/
    ├── persistence/
    │   └── (existing R2DBC adapters with added query methods)
    └── web/
        └── NetworkRestController.java  # @RestController: GET /api/network/graph, GET /api/network/faculty-stats
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
- [ ] T007 [US1] Integration test: GET /api/network/graph with no accepted matches → empty JSON arrays

### Implementation for User Story 1

- [ ] T008 [US1] Implement NetworkService.buildGraphData():
  - Query all accepted matches with student profiles (R2DBC joins)
  - Build student nodes: id=studentId, label=fullName, type=ESTUDIANTE, faculty
  - Derive faculty nodes: distinct faculties from student nodes, type=FACULTAD
  - Build match edges: source=studentA, target=studentB (or student-project), type=MATCH
  - Build faculty edges: aggregate count of matches between faculty pairs, weight=count, type=FACULTY_CONNECTION
  - Map weight to 3 intensity levels: BAJA (1-2), MEDIA (3-5), ALTA (6+)
  - Build GraphDataDto, return as JSON
- [ ] T009 [US1] Implement NetworkRestController.getGraph (GET /api/network/graph): build GraphDataDto, return as JSON, optional @RequestParam faculty filter
- [ ] T010 [US1] Implement faculty filter: when faculty param present, return only nodes/edges related to that faculty

---

## Phase 3: User Story 2 — Agregaciones por facultad (P3)

**Goal**: Endpoint JSON con estadísticas por facultad: conteo de estudiantes, proyectos, top habilidades.

**Independent Test**: GET /api/network/faculty-stats → JSON con arrays de FacultyStatsDto.

### Tests for User Story 2

- [ ] T011 [US2] Unit test: NetworkService.getFacultyAggregations → correct student counts per faculty
- [ ] T012 [US2] Unit test: NetworkService.getFacultyAggregations → top skills per faculty ordered by frequency
- [ ] T013 [US2] Integration test: GET /api/network/faculty-stats → 200 JSON

### Implementation for User Story 2

- [ ] T014 [US2] Implement NetworkService.getFacultyAggregations():
  - R2DBC query: SELECT sp.faculty, COUNT(sp), ... FROM student_profiles sp GROUP BY sp.faculty
  - Top skills: aggregate skills per faculty, count frequency, limit to top 5
  - Project count per faculty: from Project.relatedFaculty
- [ ] T015 [US2] Implement NetworkRestController.getFacultyStats (GET /api/network/faculty-stats) — build FacultyStatsDto list, return JSON

---

## Dependencies & Execution Order

- **Phase 1-2**: Depend on plan-backend-05 (Match entity with ACEPTADO status), plan-backend-02 (StudentProfile), plan-backend-03 (Project)
- **US2**: After US1 (can be parallel, but uses same service)
- **This module is P3** — implement only after all P1 modules are complete

---

## Notes

- El grafo se calcula on-the-fly (no se persiste). Para muestra piloto de 120 estudiantes el rendimiento es suficiente.
- Solo se incluyen como nodos: estudiantes y facultades (no programas, proyectos ni habilidades).
- Los nodos de estudiantes se incluyen solo si tienen al menos un match ACEPTADO (sin nodos aislados).
- Estudiantes en múltiples facultades (máx. 2): el nodo de estudiante se conecta a ambas facultades.
- La granularidad de visualización (zoom-in/zoom-out) la maneja el frontend con la librería de grafos; el backend entrega el grafo completo.
- Peso intra-facultad (matches entre estudiantes de la misma facultad): se incluye como arista con source=target=facultad, weight=count.
- Graph data served via REST endpoint GET /api/network/graph as JSON. Faculty stats at GET /api/network/faculty-stats.
- Clean code: meaningful names, small methods, SRP. No dead code, no commented-out code.
