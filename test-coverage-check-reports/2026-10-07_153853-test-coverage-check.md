# Test Coverage Check

- **Audited at:** 2026-10-07 15:38:53 +06:30 (Asia/Rangoon)
- **Scope:** Archer workspace root, `archer-api`, `archer-app`, and `archer-mobile`
- **Method:** Static repository inspection only
- **Tests executed:** No
- **Numeric coverage:** Unknown; no generated coverage artifact was found
- **Runtime status:** Not verified

## Executive assessment

`archer-api` has a well-targeted first integration suite with 13 cases covering the highest-value parts of authentication and public project discovery. The suite exercises the real Express and Prisma boundaries rather than isolated implementation details. However, all test assets and related API changes are currently uncommitted, no CI workflow was found, and the tests were not executed during this audit. They therefore provide useful intended coverage but no confirmed or enforced regression signal yet.

API coverage is partial around token expiry and concurrency, HTTP parser/internal-error behavior, health checks, and authorization middleware. The active web and mobile implementations have no repository-owned tests or test scripts. Web authentication tests should follow the approved browser-session design rather than lock in the current long-lived `localStorage` approach, which the repository already identifies for replacement.

## Test and tooling inventory

| Project | Repository-owned test assets | Runner/script | Static assessment |
| --- | --- | --- | --- |
| `archer-api` | `test/api.test.ts` (13 cases), `test/global-setup.ts`, `vitest.config.ts`, `tsconfig.test.json` | `npm test`, `npm run test:typecheck` | Meaningful integration foundation; uncommitted and runtime-unverified |
| `archer-app` | None found | No test script or direct test framework | Missing |
| `archer-mobile` | None found | No test script or direct test framework | Missing |

No CI workflow, JUnit output, LCOV file, or other existing execution/coverage artifact was found. Dependency-provided test packages and generated Prisma code were excluded from repository-owned coverage.

## Coverage map

| Important implemented behavior | Status | Evidence | Material remaining gap |
| --- | --- | --- | --- |
| API registration, normalization, password hashing, and response secrecy | Covered | `archer-api/test/api.test.ts`; `archer-api/src/routes.ts` | Concurrent duplicate creation is not covered |
| API login rejection | Covered | `archer-api/test/api.test.ts` | Successful login assertions are used through a helper but token claims/expiry are not directly checked |
| Refresh-token rotation and sequential replay rejection | Partial | `archer-api/test/api.test.ts`; `archer-api/src/auth.ts` | Expired tokens and competing simultaneous rotations are absent |
| Logout revocation | Partial | `archer-api/test/api.test.ts` | Idempotent logout and invalid/malformed bodies are absent |
| `/me` authentication and private-field exclusion | Partial | `archer-api/test/api.test.ts` | Expired, malformed, wrong-signature, and wrong-type access tokens are absent |
| Public project published-only visibility | Covered | `archer-api/test/api.test.ts`; `archer-api/src/routes.ts` | Authenticated-owner access implied by the detail query is not reachable or tested |
| Project search and category/skill/currency/budget filters | Covered | `archer-api/test/api.test.ts` | Cross-filter combinations and exact/null budget boundaries have limited proof |
| Project pagination and ordering | Covered | `archer-api/test/api.test.ts` | Empty and out-of-range pages are not asserted |
| Project response contract, currency, decimals, and UTC timestamps | Covered | `archer-api/test/api.test.ts` | Broader contract enforcement across endpoints is absent |
| Route validation and documented error envelope | Partial | `archer-api/test/api.test.ts`; `archer-api/src/app.ts` | Malformed JSON, oversized bodies, and unexpected internal failures are absent |
| Category and skill ordering | Covered | `archer-api/test/api.test.ts` | Failure envelopes are not covered |
| Health response | Missing | `archer-api/src/routes.ts` | Status, service name, and ISO timestamp are untested |
| Role authorization middleware | Missing | `archer-api/src/auth.ts` | Allowed, forbidden, and unauthenticated branches are untested; no current route consumes it |
| API test database evolution | Partial | `archer-api/test/global-setup.ts` | Setup applies one hard-coded migration rather than the complete migration history |
| Web registration and login | Missing | `archer-app/src/Register.tsx`; `archer-app/src/LocalizedApp.tsx`; `archer-app/src/api.ts` | Validation, API failures, pending state, navigation, and safe session behavior are untested |
| Web project loading, empty, error, retry, filters, and detail states | Missing | `archer-app/src/LocalizedApp.tsx`; `archer-app/src/api.ts` | Primary implemented marketplace workflow has no UI regression tests |
| Web language and theme persistence | Missing | `archer-app/src/i18n/I18nProvider.tsx`; `archer-app/src/theme/ThemeProvider.tsx` | Storage fallback, system preference, DOM attributes, catalog parity, and reload persistence are untested |
| Mobile API client and secure authentication restoration | Missing | `archer-mobile/src/api/client.ts`; `archer-mobile/src/auth/AuthContext.tsx` | Network failures, refresh restoration, invalid-session cleanup, login, and logout are untested |

## Prioritized findings

### P0 — API tests are not yet a durable regression gate

The API test file, setup, Vitest configuration, TypeScript test configuration, route hardening change, package script, and README update are all uncommitted. No CI workflow was found, and no existing result artifact proves execution.

- **Evidence:** Git status for `archer-api`; `archer-api/test/`; `archer-api/vitest.config.ts`; `archer-api/tsconfig.test.json`; `archer-api/package.json`
- **Impact:** The suite may be omitted from a future checkout or change review and currently cannot block regressions.
- **Owner:** `archer-api` and repository-maintenance owner

### P0 — Refresh rotation lacks expiry and concurrency proof

The suite verifies sequential token rotation and rejection when the old token is reused afterward. `rotateRefreshToken` reads token state, revokes it, and creates a successor in separate operations; no test proves that simultaneous callers cannot both pass the initial state check.

- **Evidence:** `archer-api/src/auth.ts`; `archer-api/test/api.test.ts`
- **Impact:** A race could create multiple successor sessions from one refresh token. Expiration behavior can also regress without detection.
- **Owner:** `archer-api`

### P0 — Active web authentication has no tests and uses a transitional storage design

Registration and login currently persist both access and refresh tokens in `localStorage`. The repository rules reject long-lived browser access-token storage as the default, and `TODO.md` explicitly calls for replacement. Tests written against the current behavior could accidentally cement a known transitional design.

- **Evidence:** `archer-app/src/Register.tsx`; `archer-app/src/LocalizedApp.tsx`; `AGENTS.md`; `TODO.md`
- **Impact:** Authentication regressions are unprotected, while premature assertions may make the safer session redesign harder.
- **Owner:** `archer-app`, coordinated with `archer-api`

### P1 — Non-public owner project-detail behavior is unreachable or undefined

`GET /projects/:id` includes an owner condition using `req.authUser`, but the route does not apply authentication or optional-auth middleware. The existing test proves anonymous draft hiding only.

- **Evidence:** `archer-api/src/routes.ts`
- **Impact:** If an owner is intended to retrieve a draft through this endpoint, their bearer token cannot populate the identity used by the query.
- **Owner:** `archer-api`; confirm the intended API contract before changing code or tests

### P1 — Malformed JSON may bypass the documented client-error envelope

The central error handler recognizes Zod errors and reads `error.statusCode`; Express JSON-parser errors commonly expose an HTTP `status` property. No test exercises malformed JSON or oversized payloads.

- **Evidence:** `archer-api/src/app.ts`; `archer-api/test/api.test.ts`
- **Impact:** Invalid request bodies may receive an inconsistent 500 response instead of a stable 4xx error envelope.
- **Owner:** `archer-api`

### P1 — Test database setup will drift when migrations grow

The global setup reads only `prisma/migrations/20260910151837_init/migration.sql`. This matches the current single-migration repository but will not apply future migrations automatically.

- **Evidence:** `archer-api/test/global-setup.ts`; `archer-api/prisma/migrations/`
- **Impact:** Future schema changes can make tests exercise an obsolete database or fail for setup reasons unrelated to behavior.
- **Owner:** `archer-api`

### P1 — Web marketplace and preference behavior are entirely uncovered

The active entry point imports `LocalizedApp.tsx`, which implements marketplace loading/error/retry/empty states, filters, details, authentication UI, language selection, and theme controls. Neither the web manifest nor the source tree contains project-owned tests.

- **Evidence:** `archer-app/src/main.tsx`; `archer-app/src/LocalizedApp.tsx`; `archer-app/src/i18n/I18nProvider.tsx`; `archer-app/src/theme/ThemeProvider.tsx`; `archer-app/package.json`
- **Impact:** The main implemented user workflow and persistent UI preferences can regress without an automated signal.
- **Owner:** `archer-app`

### P1 — Product-status documents have drifted from implementation

`TODO.md` still says the public discovery API supports a status input, while the route now forces `PUBLISHED` and the API README does not document a status parameter. `TODO.md` also describes mobile creation and secure storage as future work although an Expo repository and SecureStore-based auth context exist.

- **Evidence:** `TODO.md`; `SPEC.md`; `archer-api/src/routes.ts`; `archer-api/README.md`; `archer-mobile/src/auth/AuthContext.tsx`
- **Impact:** Agents may implement tests against stale scope or infer an incorrect public API contract.
- **Owner:** Workspace documentation owner; resolve against `SPEC.md` before changing tests

### P2 — Mobile coverage is absent despite implemented behavior

Mobile is still described as a later release slice, but code already performs project requests, error normalization, secure refresh-token restoration, session cleanup, login, and logout.

- **Evidence:** `archer-mobile/src/api/client.ts`; `archer-mobile/src/auth/AuthContext.tsx`; `archer-mobile/package.json`; `TODO.md`
- **Impact:** Early mobile session and network behavior can drift while its delivery status remains unclear.
- **Owner:** `archer-mobile`; schedule after resolving whether mobile is active scope

## Implementation handoff queue

1. **`archer-api` — establish the suite as a verified gate.** Review and commit the current API test slice, run it when authorized, capture any failures, and add it to the repository's chosen CI workflow.
2. **`archer-api` — prove refresh invariants.** Add an expired-token case and a concurrent-rotation case that requires exactly one successful successor. If the concurrency case fails, make token consumption atomic at the database/auth layer.
3. **Workspace/API owner — resolve project-status contract drift.** Decide whether public `status` filtering is removed or whether a role-aware endpoint is required, then align `TODO.md`, API documentation, implementation, and tests.
4. **`archer-api` — define project-detail authentication behavior.** Either add optional authentication with owner/non-owner cases or remove the unreachable owner branch and preserve public-only detail semantics.
5. **`archer-api` — harden boundary errors and database setup.** Cover malformed JSON, payload limits, unexpected failures, token variants, and health output; change test setup to apply the complete migration history.
6. **`archer-app` plus API owner — finalize browser session design before broad auth assertions.** Then cover registration/login validation, pending state, server errors, safe persistence/restoration, logout, expiry, and unauthorized recovery.
7. **`archer-app` — add focused marketplace and preference tests.** Cover loading/error/retry/empty/filter/detail behavior, English/Burmese catalog parity and persistence, localized dates, theme fallback/persistence, and DOM attributes.
8. **Workspace/mobile owner — resolve mobile status.** If active, add API-client and AuthContext tests first; if deferred, update status documentation so implemented scaffolding is not mistaken for release-ready behavior.

## Limitations

- No test, coverage, application, database, server, emulator, build, typecheck, or lint command was executed.
- No numeric coverage claim is possible because no existing coverage artifact was found.
- Assertions and setup were assessed statically; passing behavior, timing, database locking, cleanup, and platform behavior remain unknown.
- Unimplemented client project management, proposals, completion, cancellation, reviews, and notifications were treated as roadmap work rather than current test-coverage defects.
- The audit preserved all existing user and uncommitted changes; only this report file was created.
