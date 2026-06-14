# Feature Specification: Backend — Gestión de Perfiles de Estudiante

**Created**: 2026-06-13
**Tech Stack**: Java 21, Spring Boot, PostgreSQL, Lombok, Spring Data R2DBC
**Depends on**: `spec-backend-01-auth.md` (requiere autenticación y rol "estudiante")
**Maps to MVP**: User Story 1 — Crear y gestionar perfil de estudiante

## User Scenarios & Testing *(mandatory)*

### User Story 1 — Crear perfil de estudiante desde cero (Priority: P1)

Como estudiante autenticado, quiero crear mi perfil académico con mis datos (facultad, programa, semestre, habilidades, intereses, experiencia, herramientas, disponibilidad y rol preferido), para que el sistema pueda usarlo en el cálculo de coincidencias.

**Why this priority**: Es la base de datos del sistema. Sin perfiles de estudiante no hay datos sobre los cuales ejecutar el motor de matching ni generar recomendaciones.

**Independent Test**: Puede probarse autenticando un usuario con rol estudiante, creando un perfil con todos los campos requeridos, y verificando que los datos persisten correctamente y el perfil se marca como "completo".

**Acceptance Scenarios**:

1. **Scenario**: Creación de perfil completo
   - **Given** un usuario autenticado con rol "estudiante" sin perfil creado
   - **When** completa el formulario con nombre, facultad, programa, semestre, habilidades (lista), intereses (lista), experiencia previa, herramientas, disponibilidad, rol preferido y guarda
   - **Then** el sistema crea el perfil, lo asocia al usuario, lo marca como "completo" (is_complete = true) y confirma la creación

2. **Scenario**: Creación de perfil solo con campos obligatorios
   - **Given** un usuario autenticado con rol "estudiante" sin perfil creado
   - **When** completa únicamente nombre, facultad, programa (campos obligatorios) y guarda
   - **Then** el sistema crea el perfil pero lo marca como "incompleto" (is_complete = false) y muestra un aviso indicando qué campos adicionales mejorarán las recomendaciones

3. **Scenario**: Intento de crear un segundo perfil
   - **Given** un usuario estudiante que ya tiene un perfil creado
   - **When** intenta acceder a la funcionalidad de "crear perfil"
    - **Then** el sistema retorna el perfil existente con HTTP 200 y los datos en JSON (un usuario = un perfil)

---

### User Story 2 — Editar y actualizar perfil (Priority: P1)

Como estudiante con perfil existente, quiero modificar mis habilidades, intereses, experiencia o cualquier campo de mi perfil, para mantenerlo actualizado y recibir mejores recomendaciones.

**Why this priority**: Los perfiles deben ser editables para reflejar cambios reales (nuevas habilidades adquiridas, cambio de intereses, nuevo semestre). Un perfil estático pierde utilidad con el tiempo.

**Independent Test**: Puede probarse cargando un perfil existente, modificando uno o varios campos (ej. agregar una nueva habilidad), guardando, y verificando que los cambios se reflejan inmediatamente en consultas posteriores.

**Acceptance Scenarios**:

1. **Scenario**: Edición de campos de perfil
   - **Given** un usuario estudiante con perfil existente
   - **When** modifica su lista de habilidades, agrega "Python" y "análisis de datos", y guarda
   - **Then** el sistema actualiza el perfil, registra la fecha de actualización, y los nuevos datos están disponibles para futuros cálculos de matching

2. **Scenario**: Perfil pasa de incompleto a completo
   - **Given** un perfil marcado como incompleto (is_complete = false)
   - **When** el estudiante completa todos los campos requeridos para el perfil completo
   - **Then** el sistema actualiza is_complete = true

3. **Scenario**: Validación de campos al editar
   - **Given** un estudiante editando su perfil
   - **When** intenta guardar el perfil dejando vacío un campo obligatorio (nombre, facultad, programa)
   - **Then** el sistema rechaza la actualización con un mensaje indicando qué campos son obligatorios

---

### User Story 3 — Visualizar perfil propio y directorio (Priority: P2)

Como usuario del sistema, quiero ver mi propio perfil y consultar el directorio de perfiles de otros estudiantes (filtrable por facultad, programa o habilidad), para descubrir talento manualmente además de las recomendaciones automáticas.

**Why this priority**: El directorio aporta valor como herramienta de descubrimiento complementaria a las recomendaciones. Es menos prioritario que la creación/edición porque sin perfiles creados no hay nada que mostrar.

**Independent Test**: Puede probarse mediante requests a los endpoints REST (GET /api/profiles/me, GET /api/profiles?faculty=X, GET /api/profiles?skill=Y) con varios perfiles creados en la base de datos, y verificando que las respuestas JSON son correctas y están paginadas.

**Acceptance Scenarios**:

1. **Scenario**: Visualización del perfil propio
   - **Given** un usuario estudiante con perfil completo
    - **When** envía GET /api/profiles/me
   - **Then** el sistema muestra todos los campos de su perfil organizados por secciones (datos académicos, habilidades, intereses, experiencia, disponibilidad)

2. **Scenario**: Búsqueda en directorio por facultad
   - **Given** múltiples perfiles en la base de datos pertenecientes a distintas facultades
    - **When** un usuario envía GET /api/profiles?faculty=Ingeniería
   - **Then** el sistema devuelve únicamente los perfiles de estudiantes de esa facultad

3. **Scenario**: Búsqueda en directorio por habilidad
   - **Given** varios perfiles con distintas habilidades
    - **When** un usuario envía GET /api/profiles?skill=Python
   - **Then** el sistema devuelve los perfiles que incluyen "Python" en su lista de habilidades, ordenados alfabéticamente

4. **Scenario**: Directorio respeta privacidad
   - **Given** un usuario no autenticado
   - **When** intenta acceder al directorio de perfiles
    - **Then** el sistema retorna HTTP 401 (el directorio solo es accesible para usuarios autenticados) [NEEDS CLARIFICATION: ¿el directorio debe ser visible para todos los usuarios autenticados o solo para responsables de proyecto?]

---

### Edge Cases

- ¿Qué sucede con los matches existentes cuando un estudiante cambia sus habilidades o intereses? Los matches en estado "sugerido" o "interesado" deben recalcularse o marcarse para revisión. Los matches "confirmados" o "rechazados" no se ven afectados.
- ¿Cómo se maneja la normalización de habilidades escritas en texto libre vs. una lista predefinida? [NEEDS CLARIFICATION: ¿se usará una taxonomía controlada de habilidades, texto libre con normalización, o un sistema híbrido (autocompletado desde un catálogo + opción de texto libre)? Esto impacta directamente el motor de matching.]
- ¿Qué ocurre si un estudiante elimina su cuenta? El perfil asociado se elimina junto con la cuenta (definido en spec-backend-01-auth, FR-AUTH-008).
- ¿Un estudiante puede pertenecer a múltiples facultades o programas (doble titulación)? [NEEDS CLARIFICATION: el modelo actual asume una sola facultad/programa. Si se requiere soportar múltiples, la relación cambia de 1:1 a 1:N en esos campos.]

## Requirements *(mandatory)*

### Functional Requirements

- **FR-PROF-001**: El sistema MUST permitir crear un perfil de estudiante asociado a un usuario con rol "estudiante", con los campos: nombre completo (obligatorio), facultad (obligatorio), programa académico (obligatorio), semestre (opcional), habilidades (opcional pero recomendado), áreas de interés (opcional pero recomendado), experiencia previa/proyectos (opcional), herramientas que sabe utilizar (opcional), disponibilidad (opcional), rol preferido en equipo (opcional).
- **FR-PROF-002**: El sistema MUST marcar automáticamente el perfil como "completo" (is_complete = true) cuando todos los campos principales tienen datos, o "incompleto" (is_complete = false) cuando solo se llenan los obligatorios.
- **FR-PROF-003**: El sistema MUST permitir editar cualquier campo del perfil en cualquier momento, registrando la fecha de última actualización.
- **FR-PROF-004**: El sistema MUST validar que los campos obligatorios (nombre, facultad, programa) no estén vacíos al crear o editar el perfil.
- **FR-PROF-005**: El sistema MUST mantener una relación uno-a-uno entre User (rol estudiante) y StudentProfile. Un usuario no puede tener más de un perfil.
- **FR-PROF-006**: El sistema MUST exponer un endpoint REST GET /api/profiles con parámetros de consulta opcionales (faculty, program, skill) para consultar el directorio de perfiles con paginación. La respuesta es JSON paginado con los datos del perfil (excluyendo información sensible).
- **FR-PROF-007**: El sistema MUST mostrar un mensaje al estudiante con perfil incompleto indicando qué campos faltan para recibir mejores recomendaciones, cuando intente acceder a la sección de recomendaciones.
- **FR-PROF-008**: El sistema MUST asociar habilidades a categorías predefinidas para normalización, según la taxonomía definida en [NEEDS CLARIFICATION: taxonomía de habilidades no definida — ver spec-unimag-match-mvp.md Edge Cases. La propuesta del proyecto incluye 8 categorías: Tecnología y datos, Comunicación y divulgación, Investigación, Diseño y creatividad, Emprendimiento y gestión, Bienestar y sociedad, Ambiente y territorio, Cultura e identidad.]
- **FR-PROF-009**: El sistema MUST registrar la fecha de creación y la fecha de última actualización del perfil (requerido por FR-012 del MVP spec para soporte de desempate en recomendaciones).

### Key Entities

- **StudentProfile**: Representa el perfil académico de un estudiante. Atributos: user_id (FK a User, único), full_name, faculty, program, semester, skills (lista de habilidades, cada una opcionalmente vinculada a una SkillCategory), interests (lista de áreas de interés), prior_experience (texto), tools (lista), availability (texto/opciones), preferred_role (texto/opciones), is_complete (boolean), created_at, updated_at. La relación con User es uno-a-uno (único user_id). La relación con SkillCategory es muchos-a-muchos via tabla de unión.
- **SkillCategory**: Representa una categoría de habilidades dentro de la taxonomía. Atributos: name, description. [NEEDS CLARIFICATION: ¿las categorías son fijas (seed data cargada via Flyway migration) o administrables?] Relaciona habilidades de estudiantes con áreas de matching.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-PROF-001**: Un estudiante puede crear y completar su perfil (campos obligatorios + al menos una habilidad y un interés) en menos de 5 minutos (SC-001 del MVP spec).
- **SC-PROF-002**: El 100% de las actualizaciones de perfil se reflejan en consultas posteriores inmediatamente (sin caché stale).
- **SC-PROF-003**: El sistema rechaza el 100% de intentos de crear un segundo perfil para un mismo usuario estudiante.
- **SC-PROF-004**: El endpoint GET /api/profiles devuelve resultados filtrados en menos de 2 segundos para la muestra piloto (120 estudiantes).
