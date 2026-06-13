# AGENTS.md

## Project
UNIMAG Match — intelligent academic matching system that connects students, projects, and research groups at Universidad del Magdalena based on skills, interests, and needs.
MVP is a web app with: student profiles, project/need registration, match scoring, recommendations, and a collaboration network graph.

## Current state
No code, build system, tests, or CI yet. Tech stack and language not chosen.
The project is in planning phase with docs in `DOCS/`.

## Key docs
- `DOCS/spec-unimag-match-mvp.md` — feature spec with 4 user stories (P1–P3), entities, and edge cases. Several items marked `[NEEDS CLARIFICATION]` (auth method, data retention, skill taxonomy).
- `DOCS/UNIMAG Match.md` — full academic proposal/thesis. Long; prefer the spec for implementation.
- `DOCS/spec-template.md` — template for new feature specs.
- `DOCS/plan-template.md` — template for implementation plans.

### Backend specs (Phase 1 — spec planning)
- `DOCS/spec-backend-01-auth.md` — Auth & user management (P1, blocks all others)
- `DOCS/spec-backend-02-student-profile.md` — Student profile CRUD (P1, depends on auth)
- `DOCS/spec-backend-03-project-need.md` — Project/need CRUD (P1, depends on auth)
- `DOCS/spec-backend-04-matching-engine.md` — Matching algorithm & rankings (P1, depends on profile + project)
- `DOCS/spec-backend-05-match-interactions.md` — Match lifecycle & notifications (P1, depends on matching)
- `DOCS/spec-backend-06-network-data.md` — Collaboration graph data queries (P3, depends on matches)

## Architecture
- Monolithic with layers (no microservices, no REST API). Server-rendered web app.
- 1 Feature = 1 Spec = 1 Plan.

## Conventions
- Specs use Gherkin-style acceptance scenarios (Given/When/Then).
- Stories are prioritized P1–P3 and must be independently testable.
- Spec template requires `NEEDS CLARIFICATION` markers for unresolved decisions.

## Before writing code
- Choose tech stack and document it (the spec suggests Python, Google Forms, Excel, Power BI, Figma as prototype tools, but nothing is settled).
- Resolve all `[NEEDS CLARIFICATION]` items in the spec.
- Create an implementation plan using `DOCS/plan-template.md`.
- Update this file with build/test/lint commands once established.
