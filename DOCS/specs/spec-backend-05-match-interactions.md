# Feature Specification: Backend — Gestión de Interacciones (Match)

**Created**: 2026-06-13
**Tech Stack**: Java 21, Spring Boot, PostgreSQL, Lombok, Spring Data R2DBC
**Depends on**: `spec-backend-04-matching-engine.md` (requiere puntajes calculados), `spec-backend-01-auth.md`
**Maps to MVP**: User Story 2 (marcar "me interesa"), User Story 3 (notificación al responsable)

## User Scenarios & Testing *(mandatory)*

### User Story 1 — Marcar interés en un proyecto o estudiante (Priority: P1)

Como usuario del sistema (estudiante o responsable), quiero marcar "me interesa" sobre una recomendación, para iniciar una conexión con la otra parte.

**Why this priority**: Es la acción que convierte una recomendación pasiva en una interacción real. Sin esto, el sistema solo muestra puntajes pero no facilita conexiones.

**Independent Test**: Puede probarse con un estudiante y un proyecto existentes, ejecutando la acción "me interesa" desde las recomendaciones del estudiante, y verificando que se crea un registro de Match con estado "interesado" y que el responsable recibe visibilidad de ese interés.

**Acceptance Scenarios**:

1. **Scenario**: Estudiante marca interés en un proyecto
   - **Given** un estudiante visualizando un proyecto recomendado con 75% de coincidencia
   - **When** hace clic en "Me interesa / Match"
   - **Then** el sistema crea un Match con estado "interesado", asociando estudiante y proyecto, y notifica al responsable del proyecto

2. **Scenario**: Responsable marca interés en un estudiante
   - **Given** un responsable visualizando un estudiante recomendado para su proyecto
   - **When** hace clic en "Me interesa" sobre ese estudiante
   - **Then** el sistema crea un Match con estado "interesado" y notifica al estudiante

3. **Scenario**: No se puede marcar interés dos veces
   - **Given** un Match ya existente entre estudiante A y proyecto X (cualquier estado)
   - **When** el estudiante A intenta marcar "me interesa" nuevamente en el proyecto X
   - **Then** el sistema rechaza la acción indicando que ya existe una interacción previa con ese proyecto

---

### User Story 2 — Aceptar o rechazar un match recibido (Priority: P1)

Como usuario que recibe una notificación de interés, quiero aceptar o rechazar el match, para confirmar la conexión o descartarla.

**Why this priority**: Completa el ciclo de interacción. Sin aceptar/rechazar, los matches quedan en estado "interesado" indefinidamente sin resolución.

**Independent Test**: Con un match en estado "interesado", el receptor acepta y se verifica que el estado cambia a "aceptado". Con otro match, el receptor rechaza y se verifica el cambio a "rechazado".

**Acceptance Scenarios**:

1. **Scenario**: Aceptar un match recibido
   - **Given** un responsable que recibió un match "interesado" del estudiante A para su proyecto X
   - **When** el responsable revisa el match y hace clic en "Aceptar"
    - **Then** el sistema cambia el estado a "aceptado", notifica al estudiante A, y muestra los datos de contacto (nombre completo y correo electrónico de ambas partes)

2. **Scenario**: Rechazar un match recibido
   - **Given** un estudiante que recibió un match "interesado" del proyecto Y
   - **When** el estudiante revisa el match y hace clic en "Rechazar"
   - **Then** el sistema cambia el estado a "rechazado" y registra que este par (estudiante, proyecto) no debe ser sugerido nuevamente mientras los datos no cambien significativamente

3. **Scenario**: Match rechazado no se vuelve a sugerir
   - **Given** un match en estado "rechazado" entre estudiante A y proyecto X
   - **When** el estudiante A solicita sus recomendaciones nuevamente
   - **Then** el proyecto X no aparece en la lista de recomendaciones

4. **Scenario**: Match rechazado se vuelve a sugerir si los datos cambian
   - **Given** un match rechazado entre estudiante A y proyecto X
   - **When** el estudiante A agrega nuevas habilidades a su perfil o el proyecto X modifica sus requisitos
    - **Then** el sistema puede volver a sugerir el proyecto X al estudiante A si se produjo un cambio significativo en los datos (nuevas habilidades en el perfil del estudiante, cambios en las habilidades requeridas del proyecto, o cambio de disponibilidad)

---

### User Story 3 — Ver historial de matches y estados (Priority: P2)

Como usuario, quiero ver una lista de todos mis matches (enviados y recibidos) con su estado actual, para dar seguimiento a mis interacciones.

**Why this priority**: Es una vista de utilidad, pero el sistema funciona sin ella si las notificaciones son claras. No bloquea el flujo principal de matching.

**Independent Test**: Con varios matches en distintos estados, el usuario accede a su historial y verifica que todos aparecen con el estado correcto.

**Acceptance Scenarios**:

1. **Scenario**: Historial de matches de un estudiante
   - **Given** un estudiante con 3 matches: uno "interesado" (enviado), uno "aceptado", uno "rechazado"
   - **When** solicita `GET /api/matches`
   - **Then** el sistema retorna un JSON con los 3 matches agrupados por estado o en orden cronológico inverso

2. **Scenario**: Historial de matches de un responsable
   - **Given** un responsable con proyectos que tienen matches en distintos estados
   - **When** solicita `GET /api/matches`
   - **Then** el sistema retorna un JSON con los matches agrupados por proyecto, con el estado de cada uno

---

### Edge Cases

- **Match rechazado por ambas partes**: Ambas partes rechazan; el match permanece en estado "rechazado" y no se vuelve a sugerir.
- **Match expirado sin respuesta**: Los matches en estado "interesado" no expiran. Un match puede permanecer sin respuesta indefinidamente hasta que el receptor tome una acción (aceptar o rechazar).
- **Un usuario elimina su cuenta con matches activos**: Los matches donde participa el usuario eliminado deben anonimizarse o eliminarse (FR-AUTH-008 en spec-backend-01-auth).
- **Notificaciones**: Las notificaciones se envían tanto dentro de la plataforma (in-app, persistidas en base de datos) como por correo electrónico.
- **Cambio de estado "aceptado"**: Un match en estado "aceptado" es irreversible y constituye un estado final. No puede pasar a "rechazado" posteriormente.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-INT-001**: El sistema MUST permitir a un estudiante marcar "me interesa" sobre un proyecto recomendado mediante `POST /api/matches` con body `{studentId, projectId}`, creando un registro de Match con estado "interesado".
- **FR-INT-002**: El sistema MUST permitir a un responsable de proyecto marcar "me interesa" sobre un estudiante recomendado mediante `POST /api/matches`, creando un registro de Match con estado "interesado".
- **FR-INT-003**: El sistema MUST evitar la creación de matches duplicados entre el mismo estudiante y proyecto (máximo un Match por par).
- **FR-INT-004**: El sistema MUST permitir al receptor de un match "interesado" aceptarlo mediante `POST /api/matches/{id}/accept` (estado → "aceptado") o rechazarlo mediante `POST /api/matches/{id}/reject` (estado → "rechazado").
- **FR-INT-005**: El sistema MUST excluir de futuras recomendaciones los pares (estudiante, proyecto) que tengan un match en estado "rechazado", a menos que los datos del estudiante o del proyecto hayan cambiado significativamente. Se considera cambio significativo: modificación de habilidades del estudiante, modificación de habilidades requeridas del proyecto, o cambio de disponibilidad.
- **FR-INT-006**: El sistema MUST notificar a la parte receptora cuando se crea un nuevo match "interesado". La notificación se retorna en la respuesta JSON y se persiste en base de datos (entidad Notification). Adicionalmente, se envía notificación por correo electrónico al receptor.
- **FR-INT-007**: El sistema MUST notificar a la parte iniciadora cuando su match es aceptado o rechazado. La notificación se retorna en la respuesta JSON y se persiste en base de datos (entidad Notification). Adicionalmente, se envía notificación por correo electrónico al iniciador.
- **FR-INT-008**: El sistema MUST permitir a cada usuario ver el historial de sus matches (enviados y recibidos) mediante `GET /api/matches`, con estado y fecha de cada uno.
- **FR-INT-009**: El sistema MUST registrar la fecha y hora de cada cambio de estado del match (created_at, updated_at, responded_at).
- **FR-INT-010**: El sistema MUST aplicar el principio de supervisión humana (FR-011 del MVP spec): ningún match "aceptado" genera acciones automáticas; la conexión es una sugerencia que ambas partes deben revisar.

### Key Entities

- **Match**: Representa una interacción entre un estudiante y un proyecto. Atributos: student_id (FK a StudentProfile), project_id (FK a Project), status (enum: sugerido | interesado | aceptado | rechazado), initiated_by (enum: estudiante | responsable), match_score (porcentaje al momento de la interacción), created_at, updated_at. Restricción: unique(student_id, project_id). Las relaciones con StudentProfile y Project son muchos-a-uno.
- **Notification**: Representa una notificación in-app persistida en PostgreSQL. Atributos: user_id (FK a User, destinatario), match_id (FK a Match), type (enum: nuevo_interes | match_aceptado | match_rechazado), message (texto), is_read (boolean), created_at.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-INT-001**: Un usuario puede marcar "me interesa" en menos de 2 segundos desde que visualiza la recomendación.
- **SC-INT-002**: El sistema rechaza el 100% de intentos de crear matches duplicados para el mismo par (estudiante, proyecto).
- **SC-INT-003**: El 100% de matches en estado "rechazado" se excluyen de futuras recomendaciones (mientras los datos no cambien).
- **SC-INT-004**: El 100% de las acciones de aceptar/rechazar generan una notificación visible para la otra parte en la plataforma.
- **SC-INT-005**: La tasa de aceptación de matches (proporción de matches "interesado" que reciben respuesta) se registra como métrica para la fase piloto (meta base: ≥60% según documento UNIMAG Match).
