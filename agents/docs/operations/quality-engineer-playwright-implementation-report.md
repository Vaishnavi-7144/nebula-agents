# Quality Engineer Playwright Implementation Report

**Date:** 2026-06-06  
**Scope:** `nebula-agents/agents/quality-engineer`  
**Related product validation:** `nebula-insurance-crm`

## Summary

The Quality Engineer agent was upgraded from a broad test-automation role into a stronger full-stack quality role that explicitly covers frontend, backend, database, and AI layers. The largest improvement is making Playwright a disciplined cross-layer verification tool, not just a browser smoke-test tool.

The implementation now tells QE how to prove a critical user flow end to end:

```text
UI action -> API request/response -> database state/audit event -> UI confirmation
```

This closes a previous gap where visual or browser-level evidence could look green while API behavior, persistence, migration validity, or rollback behavior remained unverified.

## Files Updated

- `agents/quality-engineer/SKILL.md`
- `agents/quality-engineer/references/e2e-testing-guide.md`
- `agents/quality-engineer/references/testing-best-practices.md`

## What Changed

### Quality Engineer Skill

The QE skill now:

- Names database quality as an explicit first-class scope.
- Defines full-stack traceability as a core principle.
- Adds Playwright API setup/assertion guidance through `request.newContext()`.
- Adds database verification duties for persistence, rollback, migrations, audit/timeline records, workflow state, and relationship integrity.
- Adds PASS guardrails requiring database evidence when persistence behavior changes.
- Requires Playwright full-stack evidence to include browser result, API/database assertion result, and trace/screenshot paths when available.
- Expands allowed tools for `pnpm`, `npx`, and `docker`, matching real Playwright/frontend workflows.
- Keeps the skill under the framework regression line-count ceiling by moving detailed guidance into reference files.

### E2E Testing Guide

The E2E guide changed from placeholder text into concrete Playwright guidance. It now documents:

- When to use Playwright versus faster test layers.
- Full-stack test shape: arrange, authenticate, drive UI, assert API, assert database, attach artifacts.
- Frontend locator and waiting rules.
- API assertion expectations.
- Database assertion expectations.
- Evidence checklist for browser/API/database proof.

### Testing Best Practices

The testing best-practices guide now explains the layered quality bar:

- Unit tests prove pure rules.
- Component/integration tests prove UI behavior.
- Backend integration tests prove request handling and data access.
- Database tests prove migrations, transactions, and persistence.
- Playwright proves the assembled critical journey.

It also explicitly warns against using Playwright as a substitute for missing lower-layer coverage.

## Improvements After Adding Playwright Skill

### 1. Better Full-Stack Confidence

Before, QE guidance treated E2E mostly as browser-level validation. Now Playwright is positioned as a full-stack acceptance harness for critical flows. This improves confidence that a user-visible success also corresponds to a valid API response and correct persisted state.

### 2. Stronger Backend Verification From UI Flows

Playwright API assertions now connect frontend journeys to backend contract behavior. QE can verify status codes, ProblemDetails, authorization responses, response shapes, and idempotency within the same acceptance flow while still preserving backend unit/integration tests as the deeper rule coverage.

### 3. Database Behavior Is No Longer Implicit

Database verification is now explicit. QE must validate migrations, writes, soft deletes, workflow states, audit/timeline rows, tenant/ownership fields, relationship integrity, and rollback/no-partial-write behavior when persistence is part of the feature.

### 4. Evidence Is More Useful

The new evidence model asks for:

- Exact commands.
- Browser report, trace, video, or screenshot.
- API assertion output.
- Database assertion output.
- Test data IDs.
- PASS/fail/flake notes.

That makes QE verdicts easier to audit and easier for PM, DevOps, Code Reviewer, and Security roles to trust.

### 5. Less False Confidence From Visual Smoke

The guidance now states that screenshots and visual tests support styling/theme validation, but do not replace behavior tests. This is especially important for UI flows where the page looks correct but requests, cache invalidation, auth, or persistence are broken.

## Validation Performed

### Framework Validation: `nebula-agents`

All framework-owned gates passed after trimming duplicated skill detail into references:

- `python3 agents/scripts/validate-genericness.py` — passed
- `python3 agents/scripts/validate_templates.py` — passed
- `python3 agents/scripts/run-skill-regression.py` — passed
- `git diff --check` for QE files — passed

The skill regression initially failed because `SKILL.md` exceeded the 500-line maximum. The final skill is 498 lines.

### Product Test Run: `nebula-insurance-crm`

The upgraded QE guidance was exercised against the sibling product repository by running backend, frontend, Python, contract, KG, and Playwright visual checks.

Passed:

- Backend: `dotnet test engine/tests/Nebula.Tests/Nebula.Tests.csproj`
  - `414 passed`, `1 skipped`, `0 failed`
  - Coverage artifact produced at `engine/tests/Nebula.Tests/TestResults/51849a93-bd53-4250-855d-3fe7afd35701/coverage.cobertura.xml`
- Python KG tests: `python3 -m pytest scripts/kg/tests`
  - `10 passed`
- Frontend accessibility: `pnpm --dir experience test:accessibility`
  - `8 passed`
- API contract gate: `python3 planning-mds/testing/validate-nebula-api-contract.py planning-mds/api/nebula-api.yaml`
  - passed
- Visual theme Playwright: `pnpm --dir experience test:visual:theme --reporter=line`
  - `8 passed` after installing Playwright Chromium

Failed or incomplete:

- Frontend unit: `pnpm --dir experience test`
  - `238 passed`, `1 failed`
  - Failing test: `src/features/session-continuity/tests/sessionTelemetry.test.ts:98`
  - Failure: expected `fetchMock` to be called while draining deferred failure-class events.
- Frontend integration: `pnpm --dir experience test:integration`
  - `20 passed`, `1 failed`
  - Failing test: `src/pages/tests/CreateSubmissionPage.integration.test.tsx:86`
  - Failure: expected heading `Blue Horizon Manufacturing` after submission creation.
- KG validation: `python3 scripts/kg/validate.py`
  - failed because `coverage-report.yaml` is stale.
- Frontend quality gate:
  - failed because the expected visual evidence artifact is still missing at `planning-mds/operations/evidence/f0015/artifacts/playwright-report/index.html`.

Environment notes:

- The first sandboxed backend run failed because MSBuild could not create named pipes. The escalated backend run passed.
- The first sandboxed Playwright visual run could not bind the local Vite server to `127.0.0.1:4173`. The escalated run got past that.
- Playwright Chromium had to be installed before visual tests could execute.
- `experience/pnpm-workspace.yaml` appeared as a new untracked file after Playwright/pnpm tooling ran.

## Net Result

The Quality Engineer agent is now better aligned with real full-stack product validation. It can guide QE through:

- Fast frontend behavior coverage.
- Backend and API contract checks.
- Database and migration proof.
- Playwright browser evidence.
- Cross-layer acceptance evidence that ties the story, UI, API, and persisted state together.

The product test run also demonstrated why the improvement matters: the backend and visual checks passed, but unit/integration frontend failures and missing evidence artifacts remained visible instead of being hidden behind broad E2E success.

## Recommended Next Steps

1. Fix `sessionTelemetry.test.ts` so deferred telemetry drain calls `fetch` as expected or update the test if the intended behavior changed.
2. Fix `CreateSubmissionPage.integration.test.tsx` so the create flow navigates/renders the expected submission detail state.
3. Regenerate KG coverage with `python3 scripts/kg/validate.py --write-coverage-report` if the stale report is expected to be updated.
4. Wire the passing Playwright visual report into the evidence path expected by `validate-frontend-quality-gate.py`.
5. Decide whether `experience/pnpm-workspace.yaml` should be committed or removed from the working tree.
