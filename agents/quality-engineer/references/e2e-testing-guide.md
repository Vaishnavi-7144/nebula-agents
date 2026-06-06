# E2E Testing Guide

Guidance for Playwright-first end-to-end workflows across frontend, backend, and database behavior.

## Playwright Scope

Use Playwright for the small set of journeys where browser behavior must be proven together with API and persistence effects. Keep broad validation in faster suites:

- Component and UI integration tests for frontend behavior.
- Backend integration tests for domain rules, authorization, validation, and API contracts.
- Database integration tests for migrations, repositories, transactions, and persistence rules.
- Playwright for assembled critical flows, accessibility, visual smoke, API smoke, and full-stack persistence proof.

## Full-Stack Test Shape

Every critical Playwright flow should follow this shape:

1. Arrange data through API helpers, fixtures, or an isolated test database.
2. Authenticate without brittle UI login steps unless login itself is under test.
3. Drive the user flow with accessible locators such as `getByRole` and `getByLabel`.
4. Wait for the specific API response or UI state that proves the action completed.
5. Assert API-visible results with Playwright `request` or route/response checks.
6. Assert database-visible results through sanctioned test helpers or scoped SQL against unique test data.
7. Attach HTML report, trace, screenshot/video when useful, API output, and database assertion output to feature evidence.

## Frontend Rules

- Prefer role/name locators over CSS selectors.
- Avoid fixed sleeps; wait on visible state, URL changes, responses, or bounded polling helpers.
- Validate success, error, validation, empty, loading, and permission states when they changed.
- Include axe checks for changed forms, navigation, dialogs, and complex widgets.
- Use screenshots for visual/theme regression support, not as the only behavior evidence.

## Backend/API Rules

- Use Playwright `request.newContext()` for story-level setup and assertions.
- Verify status codes, OpenAPI response shape, ProblemDetails errors, authorization behavior, and idempotency where relevant.
- Do not use Playwright API tests as the only coverage for domain logic; backend integration/unit tests remain required.

## Database Rules

- Use disposable databases, Testcontainers, or isolated CI schemas for persistence assertions.
- Use unique run IDs in seeded rows so assertions cannot match stale data.
- Assert primary entity writes plus audit/timeline records, ownership/tenant columns, soft-delete flags, foreign keys, and workflow state.
- Assert rollback/no-partial-write behavior for failed validation, authorization denial, and downstream dependency failures.
- Record migration validation when schema changes are in scope.

## Evidence Checklist

For each full-stack Playwright run, capture:

- Exact command and environment.
- Story or acceptance criterion covered.
- Browser artifact path: HTML report, trace, video, or screenshot.
- API assertion output or linked API report.
- Database assertion output or linked database test report.
- Test data IDs for created tenant/user/entities.
- Final result and rerun/flake notes.
