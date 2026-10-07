# Test Coverage Check

- **Audited at:** 2026-10-07 15:34:23 +06:30 (Asia/Rangoon)
- **Scope:** Archer workspace root, `archer-api`, `archer-app`, and `archer-mobile`
- **Method:** Static repository inspection only
- **Tests executed:** No
- **Numeric coverage:** Unknown; no existing generated coverage artifact was found
- **Runtime status:** Not verified

## Executive assessment

The API now has a meaningful first integration-test slice: 13 test cases exercise authentication and public project discovery through the real Express and Prisma boundaries. The suite is strongest around registration, login failure, refresh rotation, logout revocation, `/me`, published-project visibility, discovery filters, pagination, response serialization, validation, and reference-data ordering.

Coverage remains partial because important token edge cases, concurrent rotation, middleware branches, and some HTTP failure contracts are absent. The web and mobile clients have no repository-owned tests or direct test scripts, leaving their implemented authentication, API-state, persistence, and localization behavior uncovered.

The presence and TypeScript validity of API tests do not establish that they pass. This audit did not execute them.

## Test and tooling inventory

| Project | Test assets | Runner/script | Static assessment |
| --- | --- | --- | --- |
| `archer-api` | `test/api.test.ts` (13 cases), `test/global-setup.ts`, `vitest.config.ts`, `tsconfig.test.json` | Vitest through `npm test`; test TypeScript through `npm run test:typecheck` | Meaningful integration foundation; runtime unverified |
| `archer-app` | None found | No test script or direct test framework configured | Missing |
| `archer-mobile` | None found | No test script or direct test framework configured | Missing |

Generated Prisma client files and dependency-provided test packages were excluded from repository-owned coverage.

## Coverage map

| Implemented behavior | Status | Evidence | Material gap |
| --- | --- | --- | --- |
| API registration and password hashing | Covered | `archer-api/test/api.test.ts`; `archer-api/src/routes.ts` | Concurrent duplicate registration is not covered |
| API login rejection and response secrecy | Covered | `archer-api/test/api.test.ts`; `archer-api/src/routes.ts` | Malformed/expired access tokens are not covered |
| Refresh-token rotation and replay rejection | Partial | `archer-api/test/api.test.ts`; `archer-api/src/auth.ts` | Expiry and simultaneous rotation attempts are not covered |
| Logout revocation | Partial | `archer-api/test/api.test.ts` | Idempotent logout and malformed bodies are not covered |
| `/me` authentication and private-field exclusion | Partial | `archer-api/test/api.test.ts` | Expired, wrong-type, and malformed bearer tokens are not covered |
| Public project visibility | Covered | `archer-api/test/api.test.ts`; `archer-api/src/routes.ts` | Authenticated owner access to non-public details is ambiguous and untested |
| Project search and category/skill/currency/budget filters | Covered | `archer-api/test/api.test.ts` | Multi-filter intersection and exact budget boundaries have limited proof |
| Project pagination and newest-first ordering | Covered | `archer-api/test/api.test.ts` | Empty/out-of-range pages are not asserted |
| Project response contract, currencies, decimals, and UTC dates | Covered | `archer-api/test/api.test.ts` | Error-envelope consistency across unexpected failures is not covered |
| Query validation | Partial | `archer-api/test/api.test.ts`; `archer-api/src/app.ts` | Invalid JSON, oversized JSON, and unexpected internal failures are absent |
| Health route | Missing | `archer-api/src/routes.ts` | Status, service name, and ISO timestamp contract are untested |
| Role authorization middleware | Missing | `archer-api/src/auth.ts` | No allowed/forbidden role tests; currently no mutating route consumes it |
| Web login and registration | Missing | `archer-app/src/LocalizedApp.tsx`; `archer-app/src/Register.tsx`; `archer-app/src/api.ts` | Success, validation, error, storage, and navigation behavior are untested |
| Web project loading, empty, error, retry, filters, and detail states | Missing | `archer-app/src/LocalizedApp.tsx`; `archer-app/src/api.ts` | All implemented server-state UI paths are untested |
| Web language and theme persistence | Missing | `archer-app/src/i18n/I18nProvider.tsx`; `archer-app/src/theme/ThemeProvider.tsx` | Stored preference, system fallback, DOM attributes, and catalog parity are untested |
| Mobile API client and secure session restoration | Missing | `archer-mobile/src/api/client.ts`; `archer-mobile/src/auth/AuthContext.tsx` | Network errors, refresh restoration, cleanup, login, and logout are untested |

## Prioritized findings

### P0 — API test runtime is unverified

The API suite has static structure and assertions, but this audit cannot establish schema setup, database isolation, HTTP lifecycle behavior, or passing results without execution.

- Evidence: `archer-api/test/api.test.ts`, `archer-api/test/global-setup.ts`, `archer-api/vitest.config.ts`
- Impact: A configuration or runtime integration failure could make all 13 cases ineffective despite their presence.
- Owner: `archer-api`

### P0 — Refresh-token concurrency and expiry are not covered

The current suite proves sequential rotation and replay rejection. It does not prove that two simultaneous refresh attempts cannot both succeed, and it does not exercise an expired stored token.

- Evidence: `archer-api/src/auth.ts` (`rotateRefreshToken`), `archer-api/test/api.test.ts`
- Impact: Concurrent replay could create more than one valid successor session; expiry regressions could retain invalid sessions.
- Owner: `archer-api`

### P0 — Implemented web authentication has no automated protection

Login and registration write tokens to browser storage and navigate after API success, with localized error branches. No test validates success, rejection, busy state, validation, or response handling.

- Evidence: `archer-app/src/LocalizedApp.tsx`, `archer-app/src/Register.tsx`, `archer-app/src/api.ts`, `archer-app/package.json`
- Impact: Authentication UX and token-handling regressions can reach users without an automated signal.
- Owner: `archer-app`

### P1 — Non-public owner detail behavior is unreachable or undefined

The project-detail query includes an owner condition based on `req.authUser`, but the route does not apply authentication or optional-auth middleware. Public draft hiding is tested; owner access to a draft is not.

- Evidence: `archer-api/src/routes.ts` (`GET /projects/:id`)
- Impact: If owners are intended to retrieve their drafts through this route, a valid bearer token will not populate the identity used by the query.
- Owner: `archer-api`; confirm intended contract before implementation

### P1 — API failure-envelope coverage is incomplete

Validation errors are covered, but malformed JSON, request-size rejection, and unexpected database/internal errors are not. These paths pass through Express parsing or the central error handler rather than ordinary route validation.

- Evidence: `archer-api/src/app.ts`, `archer-api/test/api.test.ts`
- Impact: Clients may receive inconsistent or overly revealing errors on uncommon but important failure paths.
- Owner: `archer-api`

### P1 — Web marketplace states and filters are untested

The active web client implements loading, API error, retry, empty results, search, category/currency filters, project cards, and unavailable detail states without automated UI coverage.

- Evidence: `archer-app/src/LocalizedApp.tsx`, `archer-app/src/api.ts`
- Impact: The primary implemented user workflow can regress even when the API remains correct.
- Owner: `archer-app`

### P1 — Language and theme persistence are untested

English/Burmese selection, DOM language updates, localized dates, light/dark selection, storage fallback, and operating-system theme fallback are all implemented but uncovered.

- Evidence: `archer-app/src/i18n/I18nProvider.tsx`, `archer-app/src/i18n/en.ts`, `archer-app/src/i18n/my.ts`, `archer-app/src/theme/ThemeProvider.tsx`
- Impact: Persistent preferences and Burmese presentation can silently drift.
- Owner: `archer-app`

### P2 — Mobile test coverage is absent

The mobile scaffold includes API requests and secure refresh-token restoration despite being listed as a later product slice. No tests or project-owned runner configuration were found.

- Evidence: `archer-mobile/src/api/client.ts`, `archer-mobile/src/auth/AuthContext.tsx`, `archer-mobile/package.json`, `SPEC.md`
- Impact: Secure storage cleanup, session restoration, and network-error behavior lack regression protection.
- Owner: `archer-mobile`; schedule when mobile becomes an active release target

## Implementation handoff queue

1. **`archer-api`: verify and stabilize the existing integration suite.** Run the authorized test command, resolve only demonstrated failures, and preserve the isolated test database design.
2. **`archer-api`: add refresh expiry and concurrent-rotation tests.** Prove an expired token is rejected and only one successor session can result from competing refresh attempts. If concurrency fails, fix rotation atomically at the owning persistence/auth layer.
3. **`archer-api`: decide the non-public project-detail contract.** If owners should read drafts, add optional authentication and tests for owner, non-owner, anonymous, malformed token, and published access. Otherwise remove the unreachable owner condition and test the public-only contract.
4. **`archer-api`: cover HTTP and error boundaries.** Add focused checks for malformed JSON, oversized payloads, unexpected internal failures, expired/malformed access tokens, and the health response.
5. **`archer-app`: establish focused component/integration tooling.** Cover registration and login success/failure first, then marketplace loading/error/retry/empty/filter/detail behavior. Mock the HTTP boundary rather than importing API internals.
6. **`archer-app`: cover preference invariants.** Add catalog-key parity plus language/theme persistence, fallback, DOM attribute, and localized-date tests.
7. **`archer-mobile`: defer broad coverage until the mobile slice is active.** When activated, start with API error normalization and secure session restoration/logout cleanup.

## Limitations

- No tests, applications, servers, databases, or coverage tools were executed.
- No numeric coverage claim is possible because no existing coverage output was found.
- Test presence was assessed statically; passing, timing, isolation, and platform behavior remain unknown.
- Roadmap-only project management, proposals, completion, reviews, and notifications were not counted as current missing-test defects.
