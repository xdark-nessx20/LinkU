# Feature Specification: UNIMAG Match – MVP (Plataforma de Conexión y Recomendación Interdisciplinaria)

**Created**: 2026-06-12

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Crear y gestionar perfil de estudiante (Priority: P1)

Como estudiante de la Universidad del Magdalena, quiero crear y completar mi perfil con mis datos académicos (facultad, programa, semestre), mis habilidades, mis áreas de interés y los proyectos o semilleros en los que he participado, para que el sistema pueda usar esta información para recomendarme conexiones relevantes y recomendarme a otros.

**Why this priority**: Es la base de todo el sistema. Sin perfiles completos y estructurados no existen datos sobre los cuales calcular coincidencias ni generar recomendaciones. Ningún otro flujo (matching, recomendaciones, red de colaboración) puede funcionar sin esto.

**Independent Test**: Puede probarse de forma independiente registrando un usuario, completando el formulario de perfil (nombre, facultad, programa, semestre, habilidades, intereses, experiencia previa, disponibilidad) y verificando que la información se guarda y se muestra correctamente en la vista de perfil. Esto entrega valor por sí solo como un "directorio de talento" navegable.

**Acceptance Scenarios**:

1. **Scenario**: Registro inicial de perfil
   - **Given** un estudiante autenticado sin perfil creado
   - **When** completa el formulario con nombre, facultad, programa, semestre, habilidades, intereses, experiencia y disponibilidad, y lo guarda
   - **Then** el sistema crea el perfil, lo marca como "completo" y lo muestra en la vista de perfil del estudiante

2. **Scenario**: Edición de perfil existente
   - **Given** un estudiante con un perfil ya creado
   - **When** edita su lista de habilidades o agrega un nuevo proyecto en el que ha trabajado
   - **Then** el sistema actualiza el perfil y refleja los cambios inmediatamente en futuras búsquedas y recomendaciones

3. **Scenario**: Perfil incompleto
   - **Given** un estudiante que solo completó campos obligatorios mínimos (nombre, facultad, programa)
   - **When** intenta acceder a recomendaciones personalizadas
   - **Then** el sistema le indica qué campos adicionales (habilidades, intereses) debe completar para recibir mejores recomendaciones

---

### User Story 2 - Match entre estudiantes y proyectos/perfiles de interés (Priority: P1)

Como estudiante, quiero ver una lista de proyectos, semilleros u otros perfiles que coincidan con mis habilidades e intereses, y poder marcar "match" (interés) en ellos, para descubrir oportunidades de colaboración interdisciplinaria.

**Why this priority**: Es el núcleo funcional de UNIMAG Match: convierte los perfiles registrados en conexiones reales. Sin esta funcionalidad, el sistema sería solo una base de datos sin valor añadido.

**Independent Test**: Puede probarse cargando un conjunto de perfiles de estudiantes y proyectos de prueba con habilidades/necesidades definidas, ejecutando el cálculo de coincidencia, y verificando que un estudiante recibe una lista de proyectos ordenada por porcentaje de match, puede ver el motivo de la recomendación y puede marcar "me interesa" en uno de ellos.

**Acceptance Scenarios**:

1. **Scenario**: Visualización de coincidencias para un estudiante
   - **Given** un estudiante con perfil completo (habilidades: Python, análisis de datos)
   - **When** accede a la sección "Proyectos recomendados para ti"
   - **Then** el sistema muestra una lista de proyectos ordenada por porcentaje de coincidencia, indicando qué habilidades coinciden

2. **Scenario**: Estudiante marca interés (match) en un proyecto
   - **Given** un estudiante visualizando un proyecto recomendado
   - **When** presiona el botón "Me interesa / Match"
   - **Then** el sistema registra el interés y notifica al responsable del proyecto/semillero que un estudiante compatible ha mostrado interés

3. **Scenario**: Explicación de la recomendación
   - **Given** un estudiante visualizando una recomendación con 75% de coincidencia
   - **When** consulta el detalle de la recomendación
   - **Then** el sistema explica de forma clara qué habilidades, intereses o experiencia coincidieron (transparencia algorítmica)

---

### User Story 3 - Recomendación de perfiles a organizaciones/proyectos (Priority: P1)

Como responsable de un proyecto, semillero, organización estudiantil o iniciativa dentro de la universidad (ej. un emprendimiento de la zona de comida, un proyecto de software que necesita backend), quiero registrar las necesidades de mi proyecto (habilidades requeridas, tipo de rol, disponibilidad) y recibir una lista de estudiantes recomendados que se ajusten a esas necesidades, para poder contactarlos y formar equipo.

**Why this priority**: Es la otra cara del matching y representa el segundo caso de uso central explícitamente solicitado (recomendar estudiantes a proyectos/organizaciones según necesidades). Junto con la Historia 2, completa el ciclo de valor bidireccional del MVP.

**Independent Test**: Puede probarse creando un "proyecto/necesidad" de prueba (ej. "puesto de perros calientes necesita cubrir turno de 2 horas, disponibilidad tarde, sin habilidades técnicas requeridas" o "proyecto de software necesita backend con experiencia en bases de datos"), ejecutando el cálculo de coincidencia contra los perfiles existentes, y verificando que se devuelve una lista de estudiantes ordenada por compatibilidad con la justificación de cada match.

**Acceptance Scenarios**:

1. **Scenario**: Registro de una necesidad/proyecto
   - **Given** un usuario con rol de "organización/responsable de proyecto"
   - **When** crea una nueva publicación de necesidad especificando habilidades requeridas, tipo de rol, área/facultad relacionada y disponibilidad esperada
   - **Then** el sistema guarda la necesidad y la incluye en el proceso de cálculo de coincidencias

2. **Scenario**: Recomendación de estudiantes para una necesidad puntual (caso "turno de perros calientes")
   - **Given** una necesidad registrada que requiere disponibilidad en horario de tarde sin habilidades técnicas específicas
   - **When** el sistema ejecuta el cálculo de coincidencias
   - **Then** se recomienda una lista de estudiantes cuya disponibilidad declarada coincide con el horario requerido, ordenados por compatibilidad

3. **Scenario**: Recomendación de estudiantes para un rol técnico (caso "proyecto de software - backend")
   - **Given** una necesidad registrada que requiere habilidades de "desarrollo backend" y "bases de datos"
   - **When** el sistema ejecuta el cálculo de coincidencias
   - **Then** se recomienda una lista de estudiantes cuyas habilidades registradas incluyen desarrollo backend o tecnologías relacionadas, mostrando el porcentaje y motivo de coincidencia

4. **Scenario**: Sin coincidencias suficientes
   - **Given** una necesidad registrada con requisitos muy específicos que ningún perfil cumple por encima de un umbral mínimo
   - **When** el sistema ejecuta el cálculo de coincidencias
   - **Then** el sistema indica que no hay coincidencias fuertes y sugiere ampliar criterios o revisar perfiles con coincidencia parcial

---

### User Story 4 - Visualización de red de colaboración (Priority: P3)

Como estudiante, coordinador o usuario del sistema, quiero ver un diagrama de red (grafo) donde los nodos representen estudiantes y facultades, y las conexiones representen interacciones de colaboración (matches realizados), para identificar qué facultades y estudiantes interactúan más entre sí y descubrir patrones de colaboración interdisciplinaria.

**Why this priority**: Aporta valor analítico y de visualización ("mapa del talento"), alineado con el componente de análisis de redes del proyecto, pero no es indispensable para el funcionamiento básico de matching y recomendación. Se considera una mejora posterior al MVP funcional principal (P1s).

**Independent Test**: Puede probarse generando un conjunto de datos de matches confirmados entre estudiantes/facultades, construyendo el grafo (nodos = estudiantes y facultades, aristas = relaciones de match) y verificando que la visualización renderiza correctamente, permite distinguir el grosor/peso de las conexiones entre facultades distintas (ej. Sistemas-Medicina) y es navegable (zoom, filtro por facultad).

**Acceptance Scenarios**:

1. **Scenario**: Visualización general de la red
   - **Given** un conjunto de matches confirmados entre estudiantes de distintas facultades
   - **When** el usuario accede a la sección "Mapa de conexiones"
   - **Then** el sistema muestra un grafo con nodos representando estudiantes y facultades, y aristas representando relaciones de colaboración

2. **Scenario**: Identificación de interacción entre facultades
   - **Given** múltiples estudiantes de la Facultad de Ingeniería que han hecho match con proyectos/estudiantes de la Facultad de Medicina
   - **When** el usuario visualiza el mapa de conexiones
   - **Then** la conexión entre los nodos de "Facultad de Ingeniería" y "Facultad de Medicina" se muestra con mayor peso/grosor proporcional al número de interacciones

3. **Scenario**: Filtro por facultad o programa
   - **Given** el mapa de conexiones generado
   - **When** el usuario filtra por una facultad específica
   - **Then** el sistema resalta o aísla los nodos y conexiones relacionados con esa facultad

---

### Edge Cases

- ¿Qué sucede cuando un estudiante no ha registrado ninguna habilidad o interés? El sistema no debe generar recomendaciones vacías sin explicación; debe indicar que el perfil necesita más información para generar coincidencias.
- ¿Cómo maneja el sistema una necesidad/proyecto que no recibe ninguna postulación ni coincidencia tras un periodo de tiempo? [NEEDS CLARIFICATION: ¿debe notificarse al responsable o sugerir ampliar criterios automáticamente?]
- ¿Qué ocurre si dos estudiantes con perfiles idénticos compiten por la misma recomendación? El orden de desempate debe estar definido y ser explicable (ej. orden alfabético, fecha de actualización del perfil, disponibilidad).
- ¿Cómo se gestiona un "match" que es rechazado por una de las partes (estudiante o responsable de proyecto)? Debe registrarse para no volver a sugerir esa combinación de forma repetitiva, salvo que cambien los datos relevantes.
- ¿Qué pasa si un estudiante elimina su cuenta o solicita el borrado de sus datos? El sistema debe eliminar o anonimizar su información de perfiles, matches y del grafo de red, conforme a los principios de protección de datos del proyecto.
- ¿Cómo se valida que las habilidades e intereses ingresados sean consistentes (texto libre vs. categorías predefinidas)? [NEEDS CLARIFICATION: ¿se usará una lista controlada/categorías predefinidas de habilidades o texto libre con normalización posterior?]
- ¿Qué sucede si un mismo estudiante pertenece a múltiples programas/facultades (ej. doble programa)? El modelo de perfil y el grafo de red deben soportar esta posibilidad o definir explícitamente que solo se admite una facultad/programa principal.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: El sistema MUST permitir a un estudiante crear una cuenta y un perfil que incluya: nombre completo, facultad, programa académico, semestre, habilidades principales, áreas de interés, experiencia previa/proyectos en los que ha trabajado, herramientas que sabe utilizar, disponibilidad y rol preferido en un equipo.
- **FR-002**: El sistema MUST permitir a un estudiante editar y actualizar su perfil en cualquier momento, reflejando los cambios en futuras recomendaciones.
- **FR-003**: El sistema MUST permitir registrar "proyectos/necesidades" (proyectos académicos, semilleros, organizaciones, vacantes puntuales como turnos) con: nombre, descripción, habilidades requeridas, tipo de rol, facultad/área relacionada y disponibilidad esperada.
- **FR-004**: El sistema MUST calcular un puntaje o porcentaje de coincidencia entre un perfil de estudiante y un proyecto/necesidad, basado en la comparación de habilidades, intereses, disponibilidad y experiencia, usando una lógica de reglas simple y explicable (sin requerir modelos de IA complejos en el MVP).
- **FR-005**: El sistema MUST mostrar a cada estudiante una lista de proyectos/necesidades recomendados, ordenada por porcentaje de coincidencia, incluyendo una explicación de los factores que generaron la recomendación (transparencia algorítmica).
- **FR-006**: El sistema MUST mostrar a cada responsable de proyecto/necesidad una lista de estudiantes recomendados, ordenada por porcentaje de coincidencia, incluyendo una explicación de los factores que generaron la recomendación.
- **FR-007**: Users (estudiantes y responsables de proyectos) MUST be able to marcar "match" / "me interesa" sobre un perfil o proyecto recomendado, generando una notificación o registro visible para la otra parte.
- **FR-008**: El sistema MUST permitir consultar y filtrar el directorio de proyectos/necesidades y de perfiles (ej. por facultad, programa o habilidad) para apoyar el descubrimiento manual además de las recomendaciones automáticas.
- **FR-009**: El sistema SHOULD (P3) generar y mostrar una visualización de red (grafo) donde los nodos sean estudiantes y facultades, y las conexiones representen relaciones de match/colaboración confirmadas, permitiendo identificar qué facultades/estudiantes interactúan más entre sí.
- **FR-010**: El sistema MUST aplicar los principios de protección de datos definidos para el proyecto: consentimiento informado en el registro, minimización de datos (solo información académica relevante, sin datos sensibles como salud, biometría o finanzas), y posibilidad de que el estudiante elimine o anonimice su información.
- **FR-011**: El sistema MUST evitar presentar las recomendaciones como decisiones definitivas: todo match generado por el sistema es una sugerencia que debe ser revisada y aceptada por ambas partes (supervisión humana).
- **FR-012**: El sistema MUST registrar la fecha de creación/actualización de cada perfil y necesidad para soportar el cálculo de coincidencias y el desempate entre recomendaciones equivalentes.
- **FR-013**: El sistema MUST authenticate users via [NEEDS CLARIFICATION: método de autenticación no especificado - ¿correo institucional UNIMAG, usuario/contraseña simple, u otro?]
- **FR-014**: El sistema MUST retain user data for [NEEDS CLARIFICATION: periodo de retención no especificado, especialmente tras la finalización de la fase piloto o el retiro voluntario de un estudiante]

### Key Entities

- **Estudiante (Perfil)**: Representa a un estudiante de pregrado. Atributos clave: nombre, facultad, programa, semestre, habilidades, áreas de interés, experiencia/proyectos previos, herramientas, disponibilidad, rol preferido. Se relaciona con Proyectos/Necesidades (a través de Matches) y con Facultad/Programa.
- **Proyecto / Necesidad**: Representa una iniciativa, semillero, organización o vacante puntual que requiere colaboradores. Atributos clave: nombre, descripción, habilidades requeridas, tipo de rol, facultad/área relacionada, disponibilidad esperada, responsable. Se relaciona con Estudiantes (a través de Matches) y con Facultad/Programa.
- **Facultad / Programa**: Representa la unidad académica a la que pertenece un estudiante o un proyecto. Usada como nodo en la visualización de red y como criterio de filtrado/recomendación.
- **Match**: Representa una relación de interés/coincidencia entre un Estudiante y un Proyecto/Necesidad (o entre dos Estudiantes). Atributos clave: porcentaje de coincidencia, factores que la explican, estado (sugerido, interesado, confirmado, rechazado), fecha.
- **Habilidad / Interés**: Representa una capacidad, herramienta o área temática que puede asociarse tanto a Estudiantes (lo que ofrecen) como a Proyectos/Necesidades (lo que requieren), siendo la base del cálculo de coincidencias.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Un estudiante puede crear y completar su perfil (todos los campos principales) en menos de 5 minutos.
- **SC-002**: El sistema genera recomendaciones de coincidencia (estudiante-proyecto y proyecto-estudiante) para el 100% de los perfiles y necesidades que tengan al menos una habilidad o interés registrado.
- **SC-003**: Al menos el 80% de los usuarios de la fase piloto (aprox. 120 estudiantes, 15 semilleros, 10 proyectos) consideran que las recomendaciones recibidas son relevantes o pertinentes según encuesta de validación.
- **SC-004**: Cada recomendación mostrada incluye una explicación comprensible de los factores de coincidencia, verificada en el 100% de los casos durante pruebas.
- **SC-005**: El sistema procesa el cálculo de coincidencias para la muestra piloto completa (120 estudiantes, 25 proyectos/necesidades) en menos de 10 segundos.
- **SC-006**: (P3) El mapa de red de colaboración refleja correctamente, en pruebas con datos simulados, al menos 3 niveles de intensidad de conexión entre facultades (baja, media, alta), validado mediante revisión manual de los datos de origen.
