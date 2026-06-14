# Feature Specification: Backend — Motor de Matching y Recomendación

**Created**: 2026-06-13
**Tech Stack**: Java 21, Spring Boot, PostgreSQL, Lombok, Spring Data R2DBC
**Depends on**: `spec-backend-02-student-profile.md`, `spec-backend-03-project-need.md` (requiere perfiles y proyectos en la base de datos)
**Maps to MVP**: User Story 2 (Match entre estudiantes y proyectos), User Story 3 (Recomendación de perfiles a organizaciones) — parte de cálculo

## User Scenarios & Testing *(mandatory)*

### User Story 1 — Cálculo de coincidencias estudiante-proyecto (Priority: P1)

Como motor del sistema, debo calcular un puntaje de compatibilidad entre cada estudiante y cada proyecto activo, basado en reglas ponderadas y explicables, para generar recomendaciones ordenadas por relevancia.

**Why this priority**: Es el núcleo funcional de UNIMAG Match. Convierte datos estáticos (perfiles y proyectos) en recomendaciones accionables. Sin esto, el sistema es solo un directorio.

**Independent Test**: Puede probarse con datos de prueba: 3 estudiantes (con habilidades definidas) y 2 proyectos (con necesidades definidas), ejecutando el cálculo, y verificando que cada par (estudiante, proyecto) recibe un puntaje con desglose de factores.

**Acceptance Scenarios**:

1. **Scenario**: Coincidencia por habilidades
   - **Given** un estudiante con habilidades ["Python", "SQL", "visualización de datos"] y un proyecto que requiere ["Python", "diseño gráfico"]
   - **When** el motor calcula la coincidencia
   - **Then** el puntaje incluye +30 puntos por "Python" (habilidad coincidente), 0 por "diseño gráfico" (no coincidente), y el factor de coincidencia explica "Habilidad Python coincide con lo requerido"

2. **Scenario**: Coincidencia por programa académico
   - **Given** un estudiante del programa "Ingeniería en Ciencia de Datos" y un proyecto que busca perfiles de ese programa
   - **When** el motor calcula la coincidencia
   - **Then** el puntaje suma +15 puntos por coincidencia de programa

3. **Scenario**: Coincidencia por área de interés
   - **Given** un estudiante con interés en "sostenibilidad" y un proyecto con área temática "sostenibilidad"
   - **When** el motor calcula la coincidencia
   - **Then** el puntaje suma +20 puntos por coincidencia de área de interés

4. **Scenario**: Coincidencia por experiencia previa relacionada
   - **Given** un estudiante con experiencia en "proyectos de bienestar estudiantil" y un proyecto sobre "bienestar estudiantil"
   - **When** el motor calcula la coincidencia
    - **Then** el puntaje suma +20 puntos por experiencia relacionada. La relación se determina por coincidencia de palabras clave y categoría temática entre la experiencia previa del estudiante y la descripción/área del proyecto.

5. **Scenario**: Coincidencia por disponibilidad compatible
   - **Given** un estudiante con disponibilidad "tarde" y un proyecto que requiere disponibilidad "tarde"
   - **When** el motor calcula la coincidencia
   - **Then** el puntaje suma +15 puntos por disponibilidad compatible

6. **Scenario**: Sin coincidencias — puntaje mínimo
   - **Given** un estudiante con habilidades ["danza", "música"] y un proyecto que requiere ["Python", "bases de datos"]
   - **When** el motor calcula la coincidencia
   - **Then** el puntaje es 0% (ningún criterio coincide)

---

### User Story 2 — Generación de ranking de recomendaciones para estudiante (Priority: P1)

Como estudiante, quiero recibir una lista de proyectos recomendados ordenada por porcentaje de coincidencia, con una explicación de los factores que contribuyeron a cada puntaje, para decidir cuáles me interesan.

**Why this priority**: Es la salida principal del motor para el lado del estudiante. Materializa la promesa central: "conectar estudiantes con oportunidades".

**Independent Test**: Con datos de prueba (1 estudiante, 5 proyectos con distintos grados de compatibilidad), llamando a GET /api/recommendations/student/{userId} y verificando que la respuesta JSON contiene: (a) el proyecto más compatible aparece primero, (b) cada recomendación incluye totalScore y scoreFactors, (c) proyectos con 0% de coincidencia no aparecen.

**Acceptance Scenarios**:

1. **Scenario**: Ranking ordenado por puntaje descendente
   - **Given** un estudiante y 4 proyectos con puntajes 90%, 60%, 45%, 0%
   - **When** el sistema genera recomendaciones para ese estudiante
   - **Then** los proyectos se muestran en orden: 90%, 60%, 45%. El proyecto con 0% se excluye.

 2. **Scenario**: Explicación de factores de coincidencia
    - **Given** una recomendación con puntaje para un estudiante
    - **When** el estudiante consulta el detalle
    - **Then** el sistema desglosa: "+30 Python (habilidad coincidente), +30 SQL (habilidad coincidente), +20 interés en sostenibilidad, +15 disponibilidad compatible, +10 programa relacionado = 105 → normalizado a 100% (porcentaje relativo)". Tener más de una habilidad coincidente aumenta el puntaje acumulativamente. Si el puntaje total excede el máximo de 100, se normaliza mostrando un porcentaje relativo (ej. 125 → 100%, 100 → 80%, 62.5 → 50%).

3. **Scenario**: Estudiante sin habilidades registradas
   - **Given** un estudiante con perfil incompleto (sin habilidades ni intereses)
   - **When** solicita recomendaciones
   - **Then** el sistema no calcula puntajes y muestra un mensaje: "Completa tu perfil con habilidades e intereses para recibir recomendaciones" (FR-PROF-007)

---

### User Story 3 — Generación de ranking de estudiantes para proyecto (Priority: P1)

Como responsable de proyecto, quiero recibir una lista de estudiantes recomendados ordenada por porcentaje de coincidencia, con explicación de factores, para contactar a los más compatibles.

**Why this priority**: Es la salida del motor para el lado del responsable. Cierra el ciclo bidireccional de recomendaciones.

**Independent Test**: Con datos de prueba (1 proyecto, 5 estudiantes), llamando a GET /api/recommendations/project/{projectId} y verificando que la respuesta JSON contiene los mismos criterios que en User Story 2 (totalScore y scoreFactors).

**Acceptance Scenarios**:

1. **Scenario**: Ranking de estudiantes para un proyecto con necesidades técnicas
   - **Given** un proyecto que requiere ["Python", "bases de datos"] y 4 estudiantes con distintos niveles de coincidencia
   - **When** el responsable consulta "Estudiantes recomendados"
   - **Then** los estudiantes se ordenan por porcentaje de coincidencia descendente, cada uno con explicación de factores

2. **Scenario**: Proyecto sin coincidencias suficientes
   - **Given** un proyecto con requisitos muy específicos y ningún estudiante supera el umbral mínimo
   - **When** el responsable consulta recomendaciones
   - **Then** el sistema indica "No se encontraron coincidencias fuertes. Considera ampliar los criterios de búsqueda." (FR-PROY-008) y sugiere reducir habilidades requeridas o ampliar facultades

---

### User Story 4 — Recalcular coincidencias ante cambios (Priority: P2)

Como motor del sistema, debo reflejar los cambios en perfiles o proyectos en los puntajes de coincidencia, para que las recomendaciones siempre se basen en datos actualizados.

**Why this priority**: Sin recálculo, los puntajes quedan obsoletos y las recomendaciones pierden relevancia. Es menos prioritario que el cálculo inicial porque el MVP puede ejecutar recálculo bajo demanda.

**Independent Test**: Se modifica una habilidad de un estudiante, se solicita recálculo, y se verifica que el nuevo puntaje refleja el cambio.

**Acceptance Scenarios**:

1. **Scenario**: Recalculo tras cambio de habilidades del estudiante
   - **Given** un estudiante cuyo perfil es editado (se agrega una nueva habilidad)
   - **When** se solicita el ranking de proyectos para ese estudiante
   - **Then** los puntajes reflejan la nueva habilidad en el cálculo

2. **Scenario**: Recalculo tras desactivación de proyecto
   - **Given** un proyecto que es marcado como inactivo
   - **When** cualquier estudiante solicita sus recomendaciones
   - **Then** el proyecto inactivo no aparece en ningún ranking

---

### Edge Cases

- **Empate entre dos estudiantes con idéntico puntaje**: El desempate es determinista y explicable. Criterios en orden: (1) fecha de actualización de perfil más reciente primero, (2) orden alfabético por nombre del estudiante.
- **Rendimiento con la muestra piloto**: El sistema debe procesar el cálculo para 120 estudiantes × 25 proyectos = 3000 comparaciones en menos de 10 segundos (SC-005 del MVP spec). El cálculo se ejecuta bajo demanda por usuario inicialmente (no precalculado periódicamente).
- **Umbral mínimo de coincidencia**: Solo se muestran recomendaciones con puntaje ≥ 50% (tras normalización). Las recomendaciones por debajo del umbral se excluyen de los rankings.
- **Habilidades con nombres equivalentes**: Se resuelve mediante la taxonomía controlada de habilidades definida en spec-backend-02-student-profile.md. Cada habilidad se asocia a una SkillCategory predefinida. La coincidencia se evalúa por habilidad exacta dentro del catálogo, no por similitud textual.
- **Ponderación de criterios**: Los puntajes definidos (+30 por habilidad, +15 programa, +20 interés, +20 experiencia, +15 disponibilidad) son los pesos iniciales para el MVP. Son configurables mediante propiedades de Spring Boot para calibración futura.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-MATCH-001**: El sistema MUST calcular un puntaje de coincidencia entre cada estudiante (con perfil) y cada proyecto activo, basado en reglas ponderadas y explicables.
- **FR-MATCH-002**: El sistema MUST usar los siguientes criterios y ponderaciones base para el cálculo:
  - Habilidad requerida por el proyecto que el estudiante posee: +30 puntos por cada habilidad coincidente (acumulativo, sin tope por cantidad de habilidades).
  - Estudiante pertenece a un programa recomendado o afín al proyecto: +15 puntos.
  - Estudiante comparte área de interés con la temática del proyecto: +20 puntos.
  - Estudiante tiene experiencia previa relacionada con el proyecto: +20 puntos.
  - Estudiante tiene disponibilidad compatible con la requerida: +15 puntos.
  - Si el puntaje total excede 100, se normaliza a un porcentaje relativo (ej. puntaje 125 se muestra como 100%, otros puntajes se escalan proporcionalmente).
- **FR-MATCH-003**: El sistema MUST generar, para cada recomendación, un desglose de factores que explique el puntaje (transparencia algorítmica — FR-005 y FR-006 del MVP spec). Ejemplo: "+30 Python, +20 sostenibilidad = 50%".
- **FR-MATCH-004**: El sistema MUST generar rankings ordenados por puntaje descendente en los siguientes endpoints REST: (a) GET /api/recommendations/student/{userId} → lista de proyectos, (b) GET /api/recommendations/project/{projectId} → lista de estudiantes. Las respuestas son JSON con totalScore y scoreFactors por cada recomendación.
- **FR-MATCH-005**: El sistema MUST excluir de los rankings los proyectos inactivos y los perfiles de estudiantes que hayan sido eliminados.
- **FR-MATCH-006**: El sistema MUST excluir de los rankings los matches en estado "rechazado" entre el mismo estudiante y proyecto, siempre que los datos no hayan cambiado (ver spec-backend-05-match-interactions.md).
- **FR-MATCH-007**: El sistema MUST indicar cuando un perfil de estudiante no tiene suficientes datos (sin habilidades ni intereses) y no generar recomendaciones en ese caso, mostrando un mensaje que invite a completar el perfil.
- **FR-MATCH-008**: El sistema SHOULD permitir que las ponderaciones de los criterios de matching sean configurables mediante propiedades de Spring Boot (application.properties). Para el MVP, se usarán valores fijos: +30/+15/+20/+20/+15.
- **FR-MATCH-009**: El sistema MUST procesar el cálculo de coincidencias para la muestra piloto (120 estudiantes × 25 proyectos = 3000 pares) en menos de 10 segundos usando Java 21 y consultas optimizadas de Spring Data R2DBC sobre PostgreSQL.
- **FR-MATCH-010**: El sistema MUST aplicar criterios de desempate deterministas y explicables cuando dos recomendaciones tengan idéntico puntaje.

### Key Entities

- **MatchScore**: Representa el resultado calculado de compatibilidad entre un estudiante y un proyecto. Atributos: student_id (FK), project_id (FK), total_score (puntaje bruto, puede superar 100), normalized_score (porcentaje relativo 0-100), skill_score, program_score, interest_score, experience_score, availability_score, score_factors (desglose JSON explicando cada factor, almacenado como JSONB en PostgreSQL), calculated_at. Se persiste en base de datos y se recalcula bajo demanda ante cambios en perfil o proyecto.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-MATCH-001**: El sistema genera un puntaje de coincidencia para el 100% de los pares (estudiante con perfil completo, proyecto activo) — SC-002 del MVP spec.
- **SC-MATCH-002**: Cada recomendación incluye un desglose explicativo de factores en el 100% de los casos — SC-004 del MVP spec.
- **SC-MATCH-003**: El cálculo completo para la muestra piloto (120 estudiantes × 25 proyectos = 3000 pares) se completa en menos de 10 segundos — SC-005 del MVP spec.
- **SC-MATCH-004**: Al menos el 80% de los usuarios de la fase piloto consideran que las recomendaciones recibidas son relevantes — SC-003 del MVP spec.
- **SC-MATCH-005**: Al modificar un perfil o proyecto, los nuevos puntajes reflejan el cambio en la siguiente solicitud de recomendaciones (consistencia de datos).
