# Feature Specification: Backend — Datos para Red de Colaboración

**Created**: 2026-06-13
**Priority**: P3
**Tech Stack**: Java 21, Spring Boot, PostgreSQL, Lombok, Spring Data R2DBC
**Depends on**: `spec-backend-05-match-interactions.md` (requiere matches confirmados para construir el grafo)
**Maps to MVP**: User Story 4 — Visualización de red de colaboración

## User Scenarios & Testing *(mandatory)*

### User Story 1 — Generar datos agregados del grafo de colaboración (Priority: P3)

Como backend, debo proveer los datos agregados necesarios para construir un grafo de colaboración: nodos (estudiantes, facultades), aristas (conexiones basadas en matches confirmados) y pesos de conexión (intensidad basada en cantidad de matches), para que el frontend pueda renderizar la visualización de red.

**Why this priority**: Aporta valor analítico y de visualización alineado con el componente de análisis de redes del proyecto, pero no bloquea el funcionamiento básico de matching. Es una mejora posterior al MVP funcional principal (P1s).

**Independent Test**: Con un conjunto de matches "aceptados" entre estudiantes de distintas facultades, se ejecuta la query de datos del grafo y se verifica que devuelve los nodos y aristas correctos, con pesos proporcionales al número de matches.

**Acceptance Scenarios**:

1. **Scenario**: Generación del grafo con matches confirmados
   - **Given** 3 estudiantes (2 de Ingeniería, 1 de Medicina) con matches "aceptados" entre ellos
   - **When** el sistema consulta los datos del grafo
   - **Then** el backend devuelve: 3 nodos de estudiante, 2 nodos de facultad (Ingeniería, Medicina), aristas entre los estudiantes que hicieron match, y aristas inter-facultad con peso = 1 (un solo match entre facultades)

2. **Scenario**: Peso de conexión entre facultades proporcional al número de matches
   - **Given** 10 matches "aceptados" entre estudiantes de Ingeniería y Medicina, y 2 matches entre Ingeniería y Derecho
   - **When** se consultan los datos del grafo
   - **Then** la arista Ingeniería-Medicina tiene peso 10 (alta intensidad) y la arista Ingeniería-Derecho tiene peso 2 (baja intensidad)

3. **Scenario**: Filtro del grafo por facultad
   - **Given** el grafo completo con múltiples facultades
   - **When** se filtran los datos por "Facultad de Ingeniería"
   - **Then** el backend devuelve solo los nodos y aristas relacionados con Ingeniería

4. **Scenario**: Grafo sin datos suficientes
   - **Given** el sistema sin matches en estado "aceptado" (solo sugeridos o interesados)
   - **When** se consultan los datos del grafo
   - **Then** el backend devuelve una estructura vacía o un mensaje indicando que no hay suficientes conexiones confirmadas para generar el grafo

---

### User Story 2 — Agregaciones por facultad y programa (Priority: P3)

Como backend, debo proveer datos agregados por facultad (número de estudiantes, número de matches, habilidades más frecuentes) para enriquecer la visualización y los dashboards.

**Why this priority**: Datos complementarios para el dashboard. No bloquea la funcionalidad principal.

**Independent Test**: Con datos de prueba de varias facultades, se consultan las agregaciones y se verifica que los conteos y frecuencias son correctos.

**Acceptance Scenarios**:

1. **Scenario**: Conteo de estudiantes por facultad
   - **Given** 50 estudiantes de Ingeniería, 30 de Medicina, 20 de Derecho
   - **When** se consulta la agregación por facultad
   - **Then** el backend devuelve: {Ingeniería: 50, Medicina: 30, Derecho: 20}

2. **Scenario**: Habilidades más frecuentes
   - **Given** perfiles de estudiantes con diversas habilidades
   - **When** se consultan las habilidades más frecuentes
   - **Then** el backend devuelve las top-N habilidades ordenadas por frecuencia descendente

---

### Edge Cases

- **Estudiante en múltiples programas/facultades**: [NEEDS CLARIFICATION: el modelo actual asume una sola facultad/programa por estudiante. Si se soportan múltiples, el grafo debe reflejar membresía múltiple.]
- **Matches entre estudiantes de la misma facultad**: Deben reflejarse como conexiones intra-facultad (autoconexiones o peso interno), no como conexiones entre facultades distintas.
- **Un estudiante o facultad sin conexiones**: Debe aparecer como nodo aislado o no aparecer, según se defina. [NEEDS CLARIFICATION: ¿nodos aislados visibles u ocultos?]
- **Rendimiento con grafos grandes**: La muestra piloto es pequeña (120 estudiantes, ~5 facultades). Para escalar, el backend debe limitar el número de nodos/aristas devueltos. [NEEDS CLARIFICATION: ¿paginación o truncamiento? No necesario para MVP.]

## Requirements *(mandatory)*

### Functional Requirements

- **FR-NET-001**: El sistema MUST proveer una consulta que devuelva los nodos y aristas del grafo de colaboración basado en matches en estado "aceptado".
- **FR-NET-002**: El sistema MUST representar como nodos: estudiantes y facultades. [NEEDS CLARIFICATION: ¿también programas, proyectos, habilidades como nodos adicionales? El documento UNIMAG Match menciona todos estos como posibles nodos.]
- **FR-NET-003**: El sistema MUST representar como aristas: relaciones de match entre estudiantes (o entre estudiante y proyecto), y calcular el peso de conexión entre facultades como la cantidad de matches "aceptados" entre estudiantes de dichas facultades.
- **FR-NET-004**: El sistema MUST soportar filtro por facultad, devolviendo solo los nodos y aristas relacionados con la facultad seleccionada.
- **FR-NET-005**: El sistema SHOULD reflejar al menos 3 niveles de intensidad de conexión entre facultades (baja, media, alta) basados en los pesos calculados (SC-006 del MVP spec).
- **FR-NET-006**: El sistema MUST exponer un endpoint REST GET /api/network/graph que devuelve los datos del grafo en formato JSON con arrays de nodos y aristas. La respuesta es consumida directamente por el frontend React.
- **FR-NET-007**: El sistema MUST excluir del grafo a estudiantes y proyectos eliminados o anonimizados.
- **FR-NET-008**: El sistema SHOULD proveer agregaciones complementarias via GET /api/network/faculty-stats: conteo de estudiantes por facultad, conteo de proyectos por facultad, y top-N habilidades más frecuentes.

### Key Entities

- **GraphData**: Representa la estructura del grafo. No es una entidad persistente sino el resultado de una consulta agregada. Contiene: nodes (array de {id, label, type: estudiante|facultad, faculty?}), edges (array de {source, target, weight, type: match|faculty_connection}).
- **FacultyAggregation**: Datos agregados por facultad: faculty_name, student_count, project_count, top_skills (array).

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-NET-001**: El grafo refleja correctamente, con datos simulados, al menos 3 niveles de intensidad de conexión entre facultades (baja/media/alta), validado mediante revisión manual de los datos de origen (SC-006 del MVP spec).
- **SC-NET-002**: La consulta de datos del grafo para la muestra piloto (120 estudiantes, 25 proyectos, ~5 facultades) se completa en menos de 3 segundos.
- **SC-NET-003**: El filtro por facultad reduce correctamente el grafo a los nodos y aristas relevantes en el 100% de las consultas.
- **SC-NET-004**: Los datos devueltos por el backend permiten al frontend renderizar el grafo sin procesamiento adicional significativo (formato directo).
