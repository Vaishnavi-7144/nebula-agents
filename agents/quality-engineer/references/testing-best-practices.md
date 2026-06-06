# Testing Best Practices

Guidance for test strategy, coverage, and cross-layer quality evidence.

## Pyramid Discipline

Use the fastest reliable layer for each risk:

- Unit tests prove pure business rules and edge cases.
- Component/integration tests prove UI behavior and API mocking paths.
- Backend integration tests prove real request handling, authorization, validation, and data access.
- Database tests prove migrations, persistence, transactions, and query behavior.
- Playwright proves the assembled user journey across browser, API, and persistence for critical flows.

Do not expand Playwright coverage to compensate for missing lower-layer tests. A passing browser journey is strong acceptance evidence, but it is too slow and coarse to replace targeted unit/integration coverage.

## Frontend Quality Bar

- Test user behavior with React Testing Library and MSW before relying on E2E.
- Verify loading, empty, error, validation, and recovery states.
- Assert accessibility semantics and keyboard paths for changed interactive UI.
- Use Playwright screenshots for visual/theme smoke evidence when layout or styling changes.

## Backend Quality Bar

- Cover domain rules with xUnit and Shouldly.
- Cover API behavior with WebApplicationFactory or the repository-standard integration harness.
- Validate OpenAPI-aligned responses, ProblemDetails, authorization, and audit behavior.
- Use Playwright request assertions only to connect API behavior to an E2E journey or perform lightweight smoke checks.

## Database Quality Bar

- Run database tests against Testcontainers or disposable databases.
- Validate migrations when schema changes are present.
- Assert transaction boundaries and rollback behavior.
- Verify audit/timeline events and tenant/ownership fields for mutations.
- Keep test data isolated with unique IDs and predictable cleanup.

## Evidence Quality

A QE verdict needs artifacts, not just a written summary. Record exact commands, result status, report paths, coverage artifacts, Playwright traces/screenshots when used, API assertion output, database assertion output, and explicit skipped-layer justification.
