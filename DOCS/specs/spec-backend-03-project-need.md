# Feature Specification: Backend — Gestión de Proyectos y Necesidades

**Created**: 2026-06-13
**Tech Stack**: Java 21, Spring Boot, PostgreSQL, Lombok, Spring Data R2DBC
**Depends on**: `spec-backend-01-auth.md` (requiere autenticación y rol "responsable")
**Maps to MVP**: User Story 3 — Recomendación de perfiles a organizaciones/proyectos (parte de registro)

## User Scenarios & Testing *(mandatory)*

### User Story 1 — Registrar un proyecto o necesidad (Priority: P1)

Como responsable de un proyecto, semillero, organización estudiantil o iniciativa (ej. puesto de comida, proyecto de software), quiero registrar mi necesidad especificando qué habilidades requiero, qué tipo de rol busco, la facultad o área relacionada y la disponibilidad esperada, para que el sistema pueda recomendarme estudiantes compatibles.

**Why this priority**: Es la entrada de datos para el lado de la demanda. Sin proyectos/necesidades registrados, el motor de matching solo tiene la mitad de la ecuación y no puede generar recomendaciones bidireccionales.

**Independent Test**: Puede probarse autenticando un usuario con rol "responsable", creando un proyecto/necesidad con todos los campos, y verificando que persiste correctamente y queda disponible para el motor de matching.

**Acceptance Scenarios**:

1. **Scenario**: Registro de un proyecto académico completo
   - **Given** un usuario autenticado con rol "responsable"
   - **When** crea un proyecto con nombre, descripción, habilidades requeridas (lista), tipo de rol, facultad relacionada, fase actual del proyecto, disponibilidad esperada y objetivo principal
   - **Then** el sistema guarda el proyecto, lo asocia al usuario responsable, y lo marca como activo para el cálculo de coincidencias

2. **Scenario**: Registro de una necesidad simple (caso "turno de perros calientes")
   - **Given** un usuario autenticado con rol "responsable"
   - **When** crea una necesidad con nombre, descripción breve, sin habilidades técnicas requeridas, con disponibilidad "tarde" y tipo de rol "colaborador"
   - **Then** el sistema guarda la necesidad correctamente sin exigir habilidades requeridas

3. **Scenario**: Registro con campos obligatorios faltantes
   - **Given** un usuario responsable en el formulario de creación
   - **When** intenta guardar un proyecto sin nombre o sin descripción
   - **Then** el sistema rechaza el registro indicando los campos obligatorios faltantes

---

### User Story 2 — Editar y gestionar proyectos propios (Priority: P1)

Como responsable de proyecto, quiero editar la información de mis proyectos (agregar o quitar habilidades requeridas, cambiar la descripción, actualizar la fase) y marcarlos como inactivos cuando ya no necesiten colaboradores, para mantener la información actualizada y no recibir recomendaciones irrelevantes.

**Why this priority**: Los proyectos evolucionan: cambian de fase, completan equipos, o modifican sus necesidades. Sin edición, las recomendaciones se basan en datos obsoletos.

**Independent Test**: Puede probarse cargando un proyecto existente, modificando sus habilidades requeridas, guardando, y verificando que los cambios se reflejan en consultas y en el motor de matching.

**Acceptance Scenarios**:

1. **Scenario**: Edición de habilidades requeridas
   - **Given** un proyecto existente que requiere "Python" y "análisis de datos"
   - **When** el responsable agrega "SQL" como nueva habilidad requerida y guarda
   - **Then** el sistema actualiza el proyecto y los futuros cálculos de matching incluyen "SQL" como criterio

2. **Scenario**: Desactivar un proyecto
   - **Given** un proyecto activo que ya completó su equipo
   - **When** el responsable marca el proyecto como inactivo
   - **Then** el sistema excluye el proyecto de futuros cálculos de coincidencias y deja de mostrarlo en recomendaciones

3. **Scenario**: Reactivar un proyecto
   - **Given** un proyecto previamente desactivado
   - **When** el responsable lo marca como activo nuevamente
   - **Then** el sistema lo reintegra al cálculo de coincidencias

---

### User Story 3 — Visualizar y filtrar proyectos (Priority: P2)

Como usuario del sistema (estudiante o responsable), quiero consultar el directorio de proyectos/necesidades con filtros por facultad, área temática o habilidad requerida, para descubrir oportunidades manualmente.

**Why this priority**: Complementa las recomendaciones automáticas con descubrimiento manual. Menos prioritario que el registro de proyectos porque sin proyectos creados no hay nada que mostrar.

**Independent Test**: Puede probarse con varios proyectos creados, llamando a los endpoints REST (GET /api/projects con query params), y verificando que las respuestas JSON son correctas.

**Acceptance Scenarios**:

1. **Scenario**: Listado de proyectos activos
   - **Given** varios proyectos en la base de datos, algunos activos y otros inactivos
   - **When** un cliente realiza una petición GET /api/projects?isActive=true
   - **Then** el sistema responde con JSON paginado que contiene únicamente los proyectos activos, ordenados por fecha de creación (más recientes primero)

2. **Scenario**: Filtro por facultad
   - **Given** proyectos de distintas facultades
   - **When** un cliente realiza una petición GET /api/projects?faculty=Ingeniería
   - **Then** el sistema responde con JSON paginado que contiene solo los proyectos de esa facultad

3. **Scenario**: Proyecto sin coincidencias suficientes
   - **Given** un proyecto activo con requisitos muy específicos para los cuales ningún estudiante tiene match superior al umbral mínimo
   - **When** el responsable consulta las recomendaciones para ese proyecto
   - **Then** el sistema indica que no hay coincidencias fuertes y sugiere ampliar criterios (FR-004 del MVP spec, Scenario 4)

---

### Edge Cases

- ¿Qué sucede con los matches asociados a un proyecto que se desactiva? Los matches en estado "sugerido" o "interesado" asociados a ese proyecto deben marcarse como "cancelados" o "expirados". Los matches "confirmados" deben notificarse al responsable para que decida.
- ¿Qué sucede con los matches asociados a un proyecto cuyas habilidades requeridas cambian? Los matches existentes deben recalcularse para reflejar los nuevos criterios. [NEEDS CLARIFICATION: ¿se recalculan automáticamente o bajo demanda?]
- ¿Qué ocurre si el responsable de un proyecto elimina su cuenta? [NEEDS CLARIFICATION: ¿los proyectos se eliminan, se transfieren a otro responsable, o quedan huérfanos? Definir en conjunto con spec-backend-01-auth.]
- ¿Un mismo usuario puede crear proyectos ilimitados? Sí, en el contexto del MVP. [NEEDS CLARIFICATION: ¿se requiere límite? No definido.]
- ¿Se permite que un estudiante (rol "estudiante") también cree proyectos? [NEEDS CLARIFICATION: el modelo actual separa roles, pero un estudiante podría también liderar un proyecto. Evaluar si se necesita rol "ambos" o permisos flexibles.]

## Requirements *(mandatory)*

### Functional Requirements

- **FR-PROY-001**: El sistema MUST permitir a un usuario con rol "responsable" crear proyectos/necesidades con: nombre (obligatorio), descripción (obligatorio), habilidades requeridas (lista, opcional), tipo de rol buscado (opcional), facultad/área relacionada (opcional), fase del proyecto (opcional), disponibilidad esperada (opcional), objetivo principal (opcional), producto/resultado esperado (opcional).
- **FR-PROY-002**: El sistema MUST permitir al responsable editar cualquier campo de sus proyectos.
- **FR-PROY-003**: El sistema MUST permitir al responsable marcar un proyecto como activo o inactivo. Los proyectos inactivos se excluyen del cálculo de coincidencias y del directorio público.
- **FR-PROY-004**: El sistema MUST asociar cada proyecto a un usuario responsable (relación uno-a-muchos: un responsable puede tener múltiples proyectos).
- **FR-PROY-005**: El sistema MUST validar que los campos obligatorios (nombre, descripción) no estén vacíos al crear o editar.
- **FR-PROY-006**: El sistema MUST registrar la fecha de creación y última actualización de cada proyecto (requerido por FR-012 del MVP spec).
- **FR-PROY-007**: El sistema MUST exponer un endpoint REST GET /api/projects con parámetros de consulta opcionales (faculty, skill, phase, isActive) para consultar el directorio de proyectos con paginación. La respuesta es JSON paginado.
- **FR-PROY-008**: El sistema MUST indicar al responsable cuando un proyecto activo no recibe coincidencias por encima del umbral mínimo [NEEDS CLARIFICATION: ¿umbral definido en spec-backend-04-matching], sugiriendo ampliar los criterios de búsqueda.

### Key Entities

- **Project**: Representa un proyecto, semillero, organización o vacante que busca colaboradores. Atributos: owner_id (FK a User), name, description, required_skills (lista, vinculada a SkillCategory), role_type, related_faculty, phase, expected_availability, main_objective, expected_output, is_active (boolean, default true), created_at, updated_at. La relación con User es muchos-a-uno (un responsable puede tener múltiples proyectos). La relación con SkillCategory es muchos-a-muchos via tabla de unión.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-PROY-001**: Un responsable puede crear un proyecto/necesidad (campos obligatorios) en menos de 3 minutos.
- **SC-PROY-002**: El 100% de proyectos inactivos se excluyen del directorio y del cálculo de coincidencias.
- **SC-PROY-003**: El endpoint GET /api/projects filtra y devuelve resultados en menos de 2 segundos para la muestra piloto (25 proyectos).
