# AGENTS.md

## Project
UNIMAG Match — intelligent academic matching system that connects students, projects, and research groups at Universidad del Magdalena based on skills, interests, and needs.
MVP is a web app with: student profiles, project/need registration, match scoring, recommendations, and a collaboration network graph.

## Current state
No code, build system, tests, or CI yet.
The project is in planning phase with docs in `DOCS/`.

## Tech Stack
- **Backend**: Java 21, Spring Boot, PostgreSQL, Lombok, Spring Security (JWT stateless), Spring Data R2DBC, Flyway
- **Frontend**: React (SPA, served as static resources from the same deployable)
- **Build**: Gradle

## Key docs
- `DOCS/specs/spec-unimag-match-mvp.md` — feature spec with 4 user stories (P1–P3), entities, and edge cases. Several items marked `[NEEDS CLARIFICATION]` (auth method, data retention, skill taxonomy).
- `DOCS/UNIMAG Match.md` — full academic proposal/thesis. Long; prefer the spec for implementation.
- `DOCS/specs/spec-template.md` — template for new feature specs.
- `DOCS/plans/plan-template.md` — template for implementation plans.

### Backend specs (Phase 1 — spec planning)
- `DOCS/specs/spec-backend-01-auth.md` — Auth & user management (P1, blocks all others)
- `DOCS/specs/spec-backend-02-student-profile.md` — Student profile CRUD (P1, depends on auth)
- `DOCS/specs/spec-backend-03-project-need.md` — Project/need CRUD (P1, depends on auth)
- `DOCS/specs/spec-backend-04-matching-engine.md` — Matching algorithm & rankings (P1, depends on profile + project)
- `DOCS/specs/spec-backend-05-match-interactions.md` — Match lifecycle & notifications (P1, depends on matching)
- `DOCS/specs/spec-backend-06-network-data.md` — Collaboration graph data queries (P3, depends on matches)

### Backend plans (Phase 2 — plan planning)
- `DOCS/plans/plan-backend-01-auth.md`
- `DOCS/plans/plan-backend-02-student-profile.md`
- `DOCS/plans/plan-backend-03-project-need.md`
- `DOCS/plans/plan-backend-04-matching-engine.md`
- `DOCS/plans/plan-backend-05-match-interactions.md`
- `DOCS/plans/plan-backend-06-network-data.md`

## Architecture
- Hexagonal (ports & adapters). Single deployable, one port. No microservices. REST API via `@RestController` returning `Mono<ResponseEntity<T>>` consumed by React SPA. No HTML templates, no Thymeleaf, no server-rendered pages.
- Authentication: JWT stateless via Spring Security `SecurityWebFilterChain`.
- Frontend: React SPA built separately, served as static resources (`/static/`) from the same Spring Boot deployable. In dev, React dev server proxies to Spring Boot via Vite.
- Spring Boot WebFlux on Netty (reactive, no Tomcat).
- Flyway for database migrations.
- 1 Feature = 1 Spec = 1 Plan.

## Conventions
- Specs use Gherkin-style acceptance scenarios (Given/When/Then).
- Stories are prioritized P1–P3 and must be independently testable.
- Spec template requires `NEEDS CLARIFICATION` markers for unresolved decisions.
- Clean Code: meaningful names, small methods, SRP, no dead code, no commented-out code, tests as documentation.

## Before writing code
- Resolve remaining `[NEEDS CLARIFICATION]` items in the specs.
- Create an implementation plan using `DOCS/plan-template.md`.
- Update this file with build/test/lint commands once established (e.g., `gradlew test`, `gradlew bootRun`).
