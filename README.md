# LinkU

**UNIMAG Match** — sistema inteligente de emparejamiento académico que conecta estudiantes, proyectos, semilleros y facultades de la Universidad del Magdalena a partir del análisis de habilidades, intereses y necesidades.

## Qué hace

- Permite a estudiantes crear perfiles con sus habilidades, intereses, experiencia y disponibilidad.
- Permite a proyectos y semilleros registrar sus necesidades de colaboración.
- Calcula coincidencias entre estudiantes y proyectos usando un motor de puntuación basado en reglas.
- Recomienda estudiantes a proyectos y proyectos a estudiantes, explicando los factores de cada match.
- Visualiza la red de colaboración interdisciplinaria entre facultades.

## Estado actual

En fase de planificación. Documentación de especificaciones en `DOCS/`. Sin código aún.

## Documentación

- `DOCS/specs/spec-unimag-match-mvp.md` — especificación funcional del MVP
- `DOCS/UNIMAG Match.md` — propuesta académica completa
- `DOCS/specs/spec-backend-01-auth.md` — especificación de autenticación y usuarios
- `DOCS/specs/spec-backend-02-student-profile.md` — especificación de perfiles de estudiante
- `DOCS/specs/spec-backend-03-project-need.md` — especificación de proyectos y necesidades
- `DOCS/specs/spec-backend-04-matching-engine.md` — especificación del motor de matching
- `DOCS/specs/spec-backend-05-match-interactions.md` — especificación de interacciones
- `DOCS/specs/spec-backend-06-network-data.md` — especificación de datos de red

## Arquitectura

Monolito con capas (sin microservicios). Una sola aplicación, un solo puerto.

- El backend (Spring Boot) sirve páginas HTML completas (Thymeleaf) e endpoints JSON internos para datos dinámicos.
- El frontend (React) consume esos endpoints JSON para funcionalidades interactivas (búsquedas, rankings, grafo).
- No hay una capa REST separada ni API externa — solo endpoints internos.

**Backend**: Java 21, Spring Boot, PostgreSQL, Lombok, Spring Security, Spring Data JPA.  
**Frontend**: React.
