# Archer Agent Guidelines

## Purpose

This file is the repository boundary and collaboration guide for Archer. It applies to the workspace root and all three independent projects:

- `archer-api` - Express API and database
- `archer-app` - React web client
- `archer-mobile` - React Native/Expo client, planned

`SPEC.md` is the product feature authority. Project `README.md` files are setup and developer documentation. Do not move implementation rules into `SPEC.md` or product requirements into README files.

## Repository Boundaries

- Treat `archer-api`, `archer-app`, and `archer-mobile` as separate repositories, even while they are developed in one workspace.
- Each project owns its own dependencies, scripts, environment examples, tests, and README.
- Clients communicate with the API through versioned HTTP contracts only.
- Do not import source files, Prisma models, database clients, or internal utilities directly across project boundaries.
- Shared types may be generated from an API contract or published as a separate package later. Do not create an implicit local shared package.
- Keep changes scoped to the project that owns the behavior. Update another project only when an API contract or user workflow requires it.

## Source Of Truth

- **Features and product behavior:** `SPEC.md`
- **Cross-project boundaries and engineering rules:** `AGENTS.md`
- **Claude-compatible entry point:** `CLAUDE.md`
- **How to install, run, test, and use a project:** that project's `README.md`
- **Implementation behavior:** source code and automated tests

When these documents disagree, stop and resolve the conflict explicitly. Do not silently invent a new product rule.

## Change Rules

- Read the relevant current files before editing; work with existing user or formatter changes.
- Fix behavior at the owning abstraction, not with a client-side workaround.
- Keep public APIs and existing conventions stable unless the feature requires a contract change.
- If an API contract changes, update the API implementation, affected clients, tests, and project READMEs in the same change.
- Avoid speculative features and unrelated refactors.
- Prefer small, testable slices that move one user workflow from API to client.
- Add or update tests for authorization, validation, lifecycle transitions, and cross-project contracts.
- Do not add payment processing, escrow, withdrawals, invoices, or transaction behavior until the product scope explicitly changes.

## API Boundary

- API routes are versioned under `/api/v1`.
- JSON responses use the documented error shape and ISO 8601 UTC timestamps.
- Validate all external input at the route boundary.
- Enforce ownership and role authorization on every mutating resource route.
- Monetary values always carry an explicit `USD` or `MMK` currency code.
- Do not convert USD and MMK values without an approved exchange-rate policy.
- Do not expose password hashes, secrets, refresh-token values, or internal database details.
- Keep access tokens short-lived and refresh tokens revocable and hashed where practical.

## Data And Seed Rules

- Prisma schema and migrations belong only to `archer-api`.
- Seed data must remain deterministic and safe to rerun.
- Development credentials must be clearly non-production and documented only in developer-facing README/setup material.
- Use decimal-safe handling for persisted monetary values.
- Preserve ownership, uniqueness, and lifecycle constraints at the database and API layers.

## Client Rules

- Web and mobile clients use the API as server state; do not duplicate server records as an alternate source of truth.
- Show loading, empty, error, unauthorized, and retry states for API-backed views.
- Preserve explicit currency labels in every budget, rate, and proposal price display.
- Keep authentication storage appropriate to each platform. Do not add long-lived browser access-token storage as a default design.
- Keep accessibility, keyboard/focus behavior, touch targets, and responsive layouts part of the feature implementation.

## Validation Before Handoff

Run the narrowest relevant checks first, then the project checks:

- API: `npm run typecheck`, `npm run build`, focused API tests, and a live endpoint smoke test when routes change.
- Web: `npm run typecheck`, `npm run lint`, `npm run build`, and focused UI tests when available.
- Mobile: documented Expo typecheck, lint, and device/simulator checks once the project exists.

Report failed checks and unrelated pre-existing failures instead of hiding them.

## Documentation Discipline

- Keep `SPEC.md` focused on user roles, capabilities, workflows, rules, scope, and acceptance criteria.
- Keep README files focused on prerequisites, setup, commands, environment variables, local URLs, demo credentials, API usage, and troubleshooting.
- Keep this file focused on boundaries, invariants, ownership, and change/validation rules.
- Do not create generated HTML or alternate copies of the specification; that creates document drift.
