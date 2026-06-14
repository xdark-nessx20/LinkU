# Feature Specification: Backend — Autenticación y Gestión de Usuarios

**Created**: 2026-06-13
**Tech Stack**: Java 21, Spring Boot, PostgreSQL, Lombok, Spring Security (JWT stateless), Spring Data R2DBC
**Depends on**: Ninguno (prerrequisito para todos los demás módulos del backend)

## User Scenarios & Testing *(mandatory)*

### User Story 1 — Registro de cuenta con consentimiento informado (Priority: P1)

Como visitante de la plataforma (estudiante o responsable de proyecto), quiero crear una cuenta aceptando los términos de uso y la política de privacidad, para poder acceder al sistema y gestionar mi perfil o mis proyectos.

**Why this priority**: Sin registro de usuarios no existe identidad dentro del sistema; ningún otro flujo (perfiles, proyectos, matching) puede iniciarse.

**Independent Test**: Puede probarse creando una cuenta nueva con email y contraseña, verificando que se persiste el usuario con los campos requeridos y el consentimiento registrado, y confirmando que sin consentimiento el registro es rechazado.

**Acceptance Scenarios**:

1. **Scenario**: Registro exitoso con consentimiento
   - **Given** un visitante en la página de registro
   - **When** completa email, contraseña, rol (estudiante/responsable), marca la casilla de consentimiento informado y envía POST /api/auth/register con el body JSON
   - **Then** el sistema crea el usuario, almacena el consentimiento con fecha y hora, y devuelve el JWT y los datos del usuario

2. **Scenario**: Registro rechazado sin consentimiento
   - **Given** un visitante en la página de registro
   - **When** completa todos los campos excepto la casilla de consentimiento informado y envía el formulario
   - **Then** el sistema rechaza el registro con un mensaje indicando que el consentimiento es obligatorio

3. **Scenario**: Registro con email duplicado
   - **Given** un usuario ya registrado con el email `a@unimagdalena.edu.co`
   - **When** otro visitante intenta registrarse con el mismo email
   - **Then** el sistema rechaza el registro con un mensaje indicando que el email ya está en uso

---

### User Story 2 — Inicio y cierre de sesión con JWT (Priority: P1)

Como usuario registrado, quiero poder iniciar sesión y cerrar sesión.

**Why this priority**: La autenticación JWT es el mecanismo que habilita el acceso autenticado a todos los endpoints de la API.

**Independent Test**: Puede probarse iniciando sesión con credenciales válidas, verificando que se devuelve un JWT, usando el token en el header Authorization para acceder a endpoints protegidos, y confirmando que sin token o con token expirado los endpoints retornan 401 Unauthorized.

**Acceptance Scenarios**:

1. **Scenario**: Inicio de sesión exitoso con JWT
   - **Given** un usuario registrado con email `a@unimagdalena.edu.co` y contraseña correcta
   - **When** envía POST /api/auth/login con {email, password}
   - **Then** el sistema devuelve HTTP 200 con un JWT en el body y los datos del usuario (id, email, rol)

2. **Scenario**: Inicio de sesión fallido por contraseña incorrecta
   - **Given** un usuario registrado
   - **When** envía POST /api/auth/login con email correcto pero contraseña incorrecta
   - **Then** el sistema retorna HTTP 401 con mensaje "Credenciales inválidas" sin revelar cuál campo es incorrecto

3. **Scenario**: Cierre de sesión (revocación de token)
   - **Given** un usuario con un JWT válido
   - **When** envía POST /api/auth/logout con el token
   - **Then** el sistema invalida el token (agregándolo a una lista de denegación) y retorna HTTP 200

4. **Scenario**: Acceso a endpoint protegido sin JWT
   - **Given** un cliente sin token JWT
   - **When** intenta acceder a un endpoint protegido (ej. GET /api/profiles)
   - **Then** el sistema retorna HTTP 401 Unauthorized

5. **Scenario**: Token JWT expirado
   - **Given** un token JWT expirado (superó el tiempo de vida configurado)
   - **When** el cliente intenta acceder a un endpoint protegido con ese token
   - **Then** el sistema retorna HTTP 401 Unauthorized

---

### User Story 3 — Recuperación de contraseña (Priority: P2)

Como usuario registrado, quiero poder restablecer mi contraseña si la olvido, para no perder el acceso a mi cuenta.

**Why this priority**: Es una funcionalidad de soporte necesaria para la usabilidad del sistema, pero no bloquea el flujo principal de matching.

**Independent Test**: Puede probarse solicitando un restablecimiento de contraseña para un email registrado, verificando que el sistema procesa la solicitud y permite establecer una nueva contraseña.

**Acceptance Scenarios**:

1. **Scenario**: Solicitud de restablecimiento de contraseña
   - **Given** un usuario registrado con email `a@unimagdalena.edu.co`
   - **When** solicita restablecer su contraseña desde la página de login
    - **Then** el sistema envía un enlace de restablecimiento al correo electrónico del usuario

2. **Scenario**: Solicitud con email no registrado
   - **Given** un email que no pertenece a ningún usuario
   - **When** se solicita restablecer contraseña para ese email
   - **Then** el sistema muestra un mensaje genérico ("Si el email está registrado, recibirás instrucciones") sin revelar si el email existe o no

---

### User Story 4 — Eliminación de cuenta y datos asociados (Priority: P1)

Como usuario registrado, quiero poder eliminar mi cuenta y todos mis datos asociados, para ejercer mi derecho de supresión conforme a los principios de protección de datos del proyecto.

**Why this priority**: Es un requisito ético y legal definido en FR-010 del MVP. La eliminación de datos debe estar disponible desde el MVP.

**Independent Test**: Puede probarse creando un usuario con perfil y matches, solicitando la eliminación de la cuenta, y verificando que el usuario, su perfil, sus matches y cualquier dato asociado son eliminados o anonimizados.

**Acceptance Scenarios**:

1. **Scenario**: Eliminación de cuenta con confirmación
   - **Given** un usuario autenticado con perfil y matches registrados
   - **When** solicita eliminar su cuenta y confirma la acción
   - **Then** el sistema elimina el usuario, su perfil, y anonimiza o elimina sus matches. El email queda liberado para un futuro registro.

2. **Scenario**: Eliminación sin confirmación
   - **Given** un usuario autenticado
   - **When** solicita eliminar su cuenta pero no confirma la acción
   - **Then** el sistema cancela la operación y la cuenta permanece activa

---

### Edge Cases

- ¿Qué sucede si un usuario intenta registrarse con un email que fue eliminado previamente? El sistema debe permitirlo ya que la eliminación es hard delete (borrado total de registros).
- ¿Cuál es el tiempo de vida del token JWT? El token de acceso (JWT) tiene una duración de 6 horas. Se emite un refresh token con duración de 7 días para renovar el acceso sin volver a iniciar sesión.
- ¿Qué ocurre si un usuario con rol "responsable" tiene proyectos activos y solicita eliminar su cuenta? El usuario debe elegir entre transferir sus proyectos a otro responsable o inactivarlos antes de proceder con la eliminación.
- ¿Se permite cambiar el rol (estudiante → responsable o viceversa) después del registro? No en el MVP; un estudiante que quiera crear un proyecto debe hacerlo desde un nuevo rol o cuenta de responsable (ver spec-backend-03-project-need).

## Requirements *(mandatory)*

### Functional Requirements

- **FR-AUTH-001**: El sistema MUST permitir el registro de usuarios con email, contraseña y rol (estudiante, responsable), exigiendo consentimiento informado obligatorio con registro de fecha y hora de aceptación.
- **FR-AUTH-002**: El sistema MUST rechazar registros con email duplicado o sin consentimiento informado.
- **FR-AUTH-003**: El sistema MUST almacenar contraseñas usando BCryptPasswordEncoder de Spring Security con sal (bcrypt).
- **FR-AUTH-004**: El sistema MUST autenticar usuarios mediante JWT stateless con Spring Security SecurityWebFilterChain, emitiendo un token firmado en el endpoint POST /api/auth/login.
- **FR-AUTH-005**: El sistema MUST proteger todos los endpoints que requieran autenticación, retornando HTTP 401 Unauthorized si el JWT está ausente, expirado o es inválido.
- **FR-AUTH-006**: El sistema MUST permitir el cierre de sesión mediante POST /api/auth/logout, invalidando el JWT en el servidor.
- **FR-AUTH-007**: El sistema MUST permitir la recuperación de contraseña mediante envío de un enlace de restablecimiento al correo electrónico del usuario.
- **FR-AUTH-008**: El sistema MUST permitir a un usuario eliminar su cuenta junto con todos sus datos asociados (perfil, matches, notificaciones), previa confirmación explícita.
- **FR-AUTH-009**: El sistema MUST validar que el email tenga un formato válido y que la contraseña cumpla con un mínimo de 8 caracteres. [NEEDS CLARIFICATION: política de complejidad adicional — ¿requerir mayúsculas, números, caracteres especiales?]
- **FR-AUTH-010**: El sistema MUST mostrar mensajes de error genéricos en login y recuperación de contraseña que no revelen si el email existe o no en la base de datos (anti-enumeración).

### Key Entities

- **User**: Representa una cuenta de acceso al sistema. Atributos: email (único), password_hash, role (enum: estudiante | responsable), consent_given (boolean), consent_date (datetime), created_at, updated_at. Se relaciona uno-a-uno con StudentProfile (si rol=estudiante) y uno-a-muchos con Project (si rol=responsable).

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-AUTH-001**: Un visitante puede completar el registro (incluyendo lectura y aceptación del consentimiento) en menos de 2 minutos.
- **SC-AUTH-002**: El sistema rechaza el 100% de intentos de registro sin consentimiento informado.
- **SC-AUTH-003**: El sistema rechaza el 100% de intentos de registro con email duplicado.
- **SC-AUTH-004**: El endpoint POST /api/auth/login retorna un JWT en menos de 2 segundos bajo carga normal.
- **SC-AUTH-005**: El 100% de endpoints protegidos retornan HTTP 401 cuando el JWT está ausente o es inválido.
- **SC-AUTH-006**: La eliminación de cuenta elimina o anonimiza el 100% de los datos asociados (perfil, matches, notificaciones) en menos de 5 segundos.
