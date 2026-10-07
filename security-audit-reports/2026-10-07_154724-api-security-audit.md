# Archer API Security Audit

- **Audit timestamp:** 2026-10-07 15:47:24 +06:30 (Asia/Rangoon)
- **Target:** `archer-api`
- **Revision inspected:** `ebfbf64f97ee2eb533905019026bc4b72ba91bb5`
- **Working tree:** Dirty; the audit includes the uncommitted `README.md`, `package.json`, `src/routes.ts`, test, and test-configuration changes present at the timestamp above.
- **Method:** Manual source/configuration review, secret-pattern scan, dependency-tree inspection, `npm audit --omit=dev --json`, and one isolated bcrypt truncation diagnostic. The API test suite was read but not executed. No source code or configuration was changed.

## Executive summary

The API has a sound early foundation: passwords are salted with bcrypt at cost 12, refresh tokens use cryptographically secure randomness and are stored only as SHA-256 hashes, access tokens expire, request bodies are bounded, database queries are parameterized through Prisma, public project queries enforce `PUBLISHED`, and Helmet plus an explicit CORS allowlist are enabled.

It is **not ready for an Internet-facing production deployment** without security hardening. The most important risks are that long-lived refresh-token values are deliberately returned to clients despite the repository requirement that they never be exposed, authentication endpoints have no automated-abuse controls, and the installed dependency tree contains a critical `proxy-addr` advisory. Refresh rotation also has a concurrency race, and accepted passwords can be silently truncated by bcrypt.

| Severity | Count |
| --- | ---: |
| Critical | 0 |
| High | 3 |
| Medium | 4 |
| Low | 3 |

## Findings

### SEC-01 — Refresh-token values are exposed in JSON responses

**Severity:** High  
**Evidence:** `src/routes.ts:203-212`, `src/routes.ts:232-240`, `src/routes.ts:247-260`; `src/config.ts:13`; `AGENTS.md:51`; `SPEC.md:75`

Registration and login return a raw refresh token in the JSON body, and refresh rotation returns the replacement token the same way. The default lifetime is 30 days. This directly conflicts with the repository rules that refresh-token values must never be exposed. In a browser client, JavaScript must receive the value and will commonly place it in memory or Web Storage; an XSS flaw can then steal a reusable 30-day credential. Hashing the database copy protects a database leak, but it does not protect the bearer value at the client boundary.

**Recommendation:** For the web client, keep the refresh token out of JavaScript by issuing it in a `Secure; HttpOnly; SameSite=Strict` (or justified `Lax`) `__Host-` cookie, and add CSRF protection appropriate to the chosen same-site deployment. Return only the short-lived access token in JSON. If the mobile client requires a bearer refresh token for OS secure storage, define a separate, explicit mobile session contract rather than weakening the browser contract. Update the API, affected clients, tests, and READMEs together because this changes the HTTP contract.

Reference: [OWASP Session Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html)

### SEC-02 — Authentication endpoints have no automated-abuse controls

**Severity:** High  
**Evidence:** `src/app.ts:8-15`; `src/routes.ts:183-276`; `src/auth.ts:13-21`

Register, login, refresh, and logout are globally reachable with no rate limit, progressive delay, per-account throttling, or upstream-control requirement. An attacker can perform credential stuffing and password spraying against login, create accounts and refresh-token rows in bulk, and consume CPU through bcrypt verification. The 1 MiB JSON limit constrains individual requests but not request frequency.

**Recommendation:** Apply endpoint-specific throttles at the trusted edge and/or API. Use both account-keyed and source-keyed limits for login, generic `429` responses with backoff, tighter limits for registration and refresh, and metrics/alerts. Do not rely only on IP limits. Before trusting `req.ip`, explicitly configure and test the exact proxy topology and remediate SEC-03.

Reference: [OWASP Authentication Cheat Sheet — Protect Against Automated Attacks](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html#protect-against-automated-attacks)

### SEC-03 — Installed dependency tree contains actionable advisories

**Severity:** High  
**Evidence:** `package-lock.json`; `package.json:16-33`; live `npm audit --omit=dev --json` and `npm explain` output on 2026-10-07

The audit reported six vulnerable package entries: one critical, four high, and one moderate. The runtime-relevant item is `proxy-addr@2.0.7`, reached through `express@5.2.1`. [GHSA-jqcg-44mw-7w3h / CVE-2026-90711](https://github.com/advisories/GHSA-jqcg-44mw-7w3h) affects versions below 2.0.8 and can make certain IPv4-mapped IPv6 trust-subnet configurations accept attacker-controlled forwarding information. The current app does not set Express `trust proxy` or use `req.ip`, so present exploitability is conditional; it becomes directly relevant when proxy-aware rate limiting, IP authorization, geolocation, or audit logging is added.

The other findings are currently on Prisma CLI/development paths rather than the SQLite API runtime:

- `deepmerge-ts@7.1.5` through `prisma -> @prisma/config` — [GHSA-ggr8-5vv4-36mx](https://github.com/advisories/GHSA-ggr8-5vv4-36mx), high.
- `mysql2@3.15.3` through the `prisma` CLI — [GHSA-3f6p-5ww8-9rcr](https://github.com/advisories/GHSA-3f6p-5ww8-9rcr), high, and [GHSA-rgwj-5xj2-c3m3](https://github.com/advisories/GHSA-rgwj-5xj2-c3m3), moderate.
- `fast-uri@3.1.7` through `prisma -> @prisma/dev -> @prisma/streams-local -> ajv` — `GHSA-hrr3-gc8f-f4qj`, moderate.

**Recommendation:** Update or override `proxy-addr` to patched version 2.0.8 or later, regenerate the lockfile, and run all API checks. Track a compatible Prisma release that resolves the CLI paths and keep Prisma tooling out of production images. Do not blindly apply npm's suggested forced Prisma downgrade to 6.19.3; review compatibility and migrations first. Add dependency auditing to CI and fail production builds on unresolved runtime high/critical advisories.

### SEC-04 — Refresh-token rotation is not atomic and permits concurrent reuse

**Severity:** Medium  
**Evidence:** `src/auth.ts:62-80`; `prisma/schema.prisma:65-74`

Rotation first reads an apparently valid token, then revokes it with a separate update, then creates a replacement. Two concurrent requests can both pass the validity check before either revocation is observed. Both can then receive different valid descendant tokens. The existing replay test covers only sequential reuse, so it does not detect this race.

**Recommendation:** Perform a conditional consume-and-create operation in one database transaction. The revocation update must match `tokenHash`, `revokedAt: null`, and an unexpired token, and rotation must proceed only when exactly one row was consumed. Add a concurrent-rotation test. Introduce a session/token-family identifier and revoke the family when reuse is detected so a stolen token that wins the race does not retain a valid descendant.

### SEC-05 — The password contract permits silent bcrypt truncation

**Severity:** Medium  
**Evidence:** `src/routes.ts:16-19`; `src/auth.ts:13-21`

The route accepts up to 100 characters, but bcrypt.js uses only the first 72 UTF-8 bytes and does not reject longer input automatically. The isolated diagnostic against the installed package confirmed that two distinct 73-byte passwords sharing the first 72 bytes authenticate against the same hash. Non-ASCII passwords can exceed 72 bytes with far fewer than 72 characters.

**Recommendation:** Prefer Argon2id for new password hashes. If retaining bcrypt, reject any registration, login, and password-change value for which `bcrypt.truncates(password)` is true, and describe the limit in bytes rather than characters. Add ASCII and multibyte boundary tests. Preserve an upgrade path for existing bcrypt hashes.

References: [bcrypt.js security considerations](https://github.com/dcodeIO/bcrypt.js/#security-considerations), [OWASP Password Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html#input-limits-of-bcrypt)

### SEC-06 — Access-token validation is weakly scoped and authorization claims can become stale

**Severity:** Medium  
**Evidence:** `src/auth.ts:9-11`, `src/auth.ts:28-41`, `src/auth.ts:83-118`, `src/auth.ts:120-132`; `src/routes.ts:278-301`

Access tokens do not carry or validate an issuer, audience, session identifier, or token identifier, and verification does not explicitly allowlist the signing algorithm. The middleware trusts the embedded email and role until expiration. A deleted/disabled user or downgraded administrator can therefore retain token claims for the remaining access-token lifetime; future routes using `requireRole` would authorize from stale state. Logout revokes only the submitted refresh token and does not invalidate an already issued access token.

Current impact is limited because `/me` re-queries the user and there are no mounted role-protected mutations yet. The risk rises as owner/admin routes are added.

**Recommendation:** Explicitly sign and verify a fixed algorithm, issuer, audience, expiration, subject, and token type. Add a session identifier or user security-version claim and validate it for privileged mutations, password changes, account disablement, and role changes. Keep the token lifetime short and define a deliberate emergency-revocation strategy. Avoid placing mutable email in the token unless it is required. The configured `JWT_REFRESH_SECRET` is currently unused because refresh tokens are opaque; remove it to avoid false assurance or document its intended future use. Require secrets with verified entropy, not only a 32-character length.

Reference: [OWASP REST Security Cheat Sheet — JWT](https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html#jwt)

### SEC-07 — Session records can grow without bound and logout is single-token only

**Severity:** Medium  
**Evidence:** `src/auth.ts:44-60`; `src/routes.ts:218-240`, `src/routes.ts:266-272`; `prisma/schema.prisma:65-74`

Every successful login creates another refresh-token row. Revoked and expired rows are retained, there is no per-user session cap, and logout revokes only the bearer value supplied by the caller. Combined with missing rate limits, repeated logins can grow the database indefinitely. Users also lack a way to revoke all sessions after compromise or password change.

**Recommendation:** Define session semantics explicitly: cap active sessions per user, expose revocation by session and “log out all,” revoke all sessions on password reset/change where appropriate, and delete expired/revoked records on a scheduled retention policy. Monitor token-table growth.

### SEC-08 — Registration response enables account enumeration

**Severity:** Low  
**Evidence:** `src/routes.ts:183-194`; `test/api.test.ts` duplicate-registration test

Registration returns `409 EMAIL_IN_USE` only when an account exists. Attackers can use this discrepancy to discover registered email addresses for phishing and credential stuffing. Login correctly uses a generic error.

**Recommendation:** If email privacy matters, use a uniform registration response and move uniqueness disclosure behind an email-verification flow. Keep timing and status behavior as consistent as practical. If product usability intentionally accepts enumeration, record that as a threat-model decision.

### SEC-09 — The project-detail path parameter is not validated at the route boundary

**Severity:** Low  
**Evidence:** `src/routes.ts:133-160`; `AGENTS.md:47`

`req.params.id` is sent directly to a parameterized Prisma query without the CUID validation applied to query-string identifiers. This is not SQL injection, but it violates the repository rule to validate all external input and allows arbitrary-length/shape identifiers to reach the database layer.

**Recommendation:** Parse path parameters with a bounded schema such as `z.string().cuid()` before querying, and add invalid-ID tests. Apply the same convention to future resource routes.

### SEC-10 — Security-relevant events are not recorded

**Severity:** Low  
**Evidence:** `prisma/schema.prisma:199-208`; no application references to `prisma.auditEvent`

An `AuditEvent` model exists, but login failures/successes, token rotation and replay, logout, role changes, and future privileged mutations are not recorded. This reduces the ability to detect credential attacks, investigate compromise, and alert on refresh-token reuse. Raw passwords and bearer tokens must never be logged.

**Recommendation:** Define a minimal security-event taxonomy and retention policy. Record actor/session identifiers, action, outcome, request correlation ID, and carefully normalized network metadata. Redact credentials and personal data, protect the log from user modification, and alert on repeated failures and token-reuse events.

## Positive controls observed

- Password hashes are not selected into public responses, and tests assert against accidental exposure.
- bcrypt cost 12 is above OWASP's minimum legacy work factor of 10.
- Refresh tokens use 48 cryptographically random bytes, are stored as hashes, expire, and can be revoked.
- Access tokens include expiration and a custom access-token type check.
- Login errors do not distinguish missing users from wrong passwords.
- Zod schemas bound email/password/name and project-query inputs; pagination is capped at 50.
- Prisma parameterization limits SQL-injection exposure in the reviewed routes.
- Public project list/detail queries prevent disclosure of non-published projects.
- Helmet is enabled, CORS is allowlisted, and JSON bodies are limited to 1 MiB.
- The global error handler hides internal messages for server errors.
- `.env` and SQLite files are ignored; the tracked-file and secret-pattern scans found no committed private keys, cloud keys, or non-example environment file.

## Validation and limitations

- `npm audit --omit=dev --json`: completed against the live npm advisory service; reported 6 vulnerable package entries (1 critical, 4 high, 1 moderate). Dependency paths were confirmed with `npm explain`.
- Bcrypt truncation diagnostic: confirmed `bcrypt.truncates(...) === true` and successful authentication of a distinct password after byte 72.
- Static secret scan: no likely committed production secret found. This was pattern-based and does not replace repository-history or hosted secret scanning.
- Tests were inspected but not run, consistent with a read-only audit. No dynamic endpoint fuzzing, concurrency test, penetration test, infrastructure/TLS review, container review, or cloud configuration review was performed.
- Only currently implemented API routes were assessed. Planned password-reset, profile mutation, project mutation, proposal, review, notification, and admin routes require authorization and lifecycle review when implemented.

## Recommended remediation order

1. Patch `proxy-addr`, decide the trusted-proxy topology, and add CI dependency gates.
2. Redesign web refresh-token transport so the value is never exposed to browser JavaScript; coordinate the client/API contract change.
3. Add auth-specific abuse controls and security-event telemetry.
4. Make refresh rotation atomic, add token-family reuse detection, and implement all-session revocation/cleanup.
5. Reject bcrypt-truncated passwords or migrate new hashes to Argon2id.
6. Harden JWT claims/verification and stale-authorization handling before adding privileged routes.
7. Close the lower-severity validation and enumeration gaps and add focused regression tests.
