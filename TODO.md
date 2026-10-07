# Archer Project TODO

Last reviewed: 2026-10-03

This file tracks implementation work across Archer's independent repositories. `SPEC.md` remains the authority for product behavior and acceptance criteria; this checklist records delivery status and the next engineering slices.

## Status Legend

- `[x]` Verified complete
- `[ ]` Not started or incomplete
- `[~]` Implemented but not yet committed or merged

## Current Focus

- [x] Initialize `archer-app` and `archer-api` as independent Git repositories.
- [x] Add persistent English and Burmese interface selection to the web client.
- [x] Add Burmese-specific typography, responsive spacing, wrapping, and Myanmar-capable font fallbacks.
- [x] Ensure Burmese typography uses `letter-spacing: normal` rather than custom tracking.
- [x] Create the project-local `.agents/skills/burmese-i18n` guidance.
- [~] Finish the light/dark theme slice on `feature/theme-mode`.
  - [x] Detect the operating-system preference on first visit.
  - [x] Add persistent theme selection and prevent a pre-render theme flash.
  - [x] Add localized, accessible toggle controls to marketplace and login screens.
  - [x] Verify light, dark, Burmese-dark, and dark-login layouts.
  - [x] Pass web typecheck, lint, build, and whitespace validation.
  - [ ] Commit the web changes and merge `feature/theme-mode` into `main`.
  - [ ] Return both repositories to `main` after the merge.
- [ ] Decide how workspace-level files (`SPEC.md`, `AGENTS.md`, `TODO.md`, and `.agents/skills`) will be version-controlled; the workspace root is not currently a Git repository.

## Release-Critical Foundations

- [ ] Complete the authentication workflow across API and web.
  - [x] Login endpoint and web login screen.
  - [x] Refresh-token API foundation.
  - [ ] Restore sessions automatically when access tokens expire.
  - [ ] Add registration and logout UI flows.
  - [ ] Add profile update, password change, and password-reset flows.
  - [ ] Replace long-lived browser token storage with the approved safer session design.
  - [ ] Add loading, validation, unauthorized, retry, and expired-session states.
- [ ] Add focused authentication tests for validation, token rotation, revocation, and unauthorized access.
- [ ] Document the finalized browser authentication/session approach in `archer-app/README.md`.

## Project Discovery

- [x] API: published-project listing and project details.
- [x] API: search plus category, skill, currency, budget, status, and pagination inputs.
- [x] Web: project search, category filter, currency filter, cards, and detail view.
- [ ] Web: expose skill and budget filters supported by the API.
- [ ] Web: add sorting controls for newest, deadline, budget, and relevance where supported.
- [ ] Web: add visible pagination instead of requesting the first 50 records.
- [ ] Show project owner and deadline in the public detail view.
- [ ] Add focused tests for loading, empty, API error, retry, filter, pagination, and unavailable-project states.

## Client Project Management

- [ ] API: create and save a draft project.
- [ ] API: edit or delete only an owned project.
- [ ] API: publish and close projects with valid lifecycle transitions.
- [ ] Web: project dashboard and draft editor.
- [ ] Web: publish, close, and delete confirmation flows.
- [ ] Test ownership, role authorization, validation, currency requirements, and invalid transitions.
- [ ] Update API and web READMEs when the project-management routes ship.

## Proposals And Active Work

- [ ] API: submit one proposal per freelancer and project.
- [ ] API: edit and withdraw eligible proposals.
- [ ] API: shortlist, reject, archive, and accept proposals as the owning client.
- [ ] API: accept at most one proposal and atomically reject other active proposals.
- [ ] API: transition an accepted project to `IN_PROGRESS` without creating payment records.
- [ ] Web: proposal submission, editing, withdrawal, comparison, and status views.
- [ ] Web: active-project summary for the client and selected freelancer.
- [ ] Test duplicate proposals, authorization, currencies, lifecycle transitions, and concurrent acceptance attempts.

## Completion, Cancellation, Reviews, And Notifications

- [ ] API: complete or cancel active work with actor, reason, and timestamp auditing.
- [ ] API: allow one immutable review per party after completion.
- [ ] API: calculate aggregate ratings from valid completed-project reviews.
- [ ] API: create notifications for proposal, project-status, review, and password-reset events.
- [ ] API: list notifications and mark them read.
- [ ] Web: completion, cancellation, review, and notification interfaces.
- [ ] Test lifecycle authorization, one-review constraints, notification creation, and read-state ownership.

## Profiles

- [ ] API: complete private profile update routes.
- [ ] API: expose public freelancer profiles without private account fields.
- [ ] Web: freelancer profile view and editor.
- [ ] Web: client profile view.
- [ ] Web: skills, portfolio, availability, rate/currency, ratings, and completed-project presentation.
- [ ] Test private-field exclusion, ownership, rate currency, portfolio validation, and aggregate counts.

## Localization, Appearance, And Accessibility

- [x] Keep USD and MMK labels explicit in all existing budget displays.
- [x] Keep user-authored and server-authored marketplace content in its original language.
- [ ] Establish and review an English-retained-term glossary for Burmese UI copy.
- [ ] Add automated catalog-key parity checks for English and Burmese.
- [ ] Add UI tests covering language and theme persistence across reloads.
- [ ] Test keyboard navigation, focus visibility, screen-reader labels, touch targets, and 200% zoom in both languages and themes.
- [ ] Audit color contrast for all interactive, error, disabled, and muted states.
- [ ] Recheck Burmese wrapping and vertical rhythm as each new screen is added.

## Mobile

- [ ] Create the independent `archer-mobile` Expo repository when mobile work begins.
- [ ] Document Expo setup, typecheck, lint, and simulator/device commands.
- [ ] Implement secure platform-appropriate authentication storage.
- [ ] Build project discovery, details, authentication, and language/theme preferences against `/api/v1`.
- [ ] Verify supported devices, accessibility, touch targets, loading, empty, error, unauthorized, and retry states.

## Cross-Project Quality

- [ ] Define or generate a versioned API contract for clients without importing API source or Prisma types.
- [ ] Add contract tests for API response shapes, error envelopes, currencies, and ISO 8601 UTC timestamps.
- [ ] Add live endpoint smoke tests for changed routes.
- [ ] Keep deterministic seed data safe to rerun as new workflows are added.
- [ ] Confirm secrets, password hashes, refresh-token values, and internal database fields never appear in responses or logs.
- [ ] Keep payment processing, escrow, invoices, withdrawals, and transaction behavior out of scope.

## Product Decisions Needed

- [ ] Decide whether users can switch between client and freelancer modes.
- [ ] Decide whether project budgets are fixed-price, hourly, or both.
- [ ] Decide whether first-release proposals require attachments and choose storage if needed.
- [ ] Decide whether unauthenticated visitors can browse public projects.
- [ ] Decide which languages and regions follow English and Burmese.
- [ ] Decide whether admin moderation screens are part of the first release.
- [ ] Decide whether push notifications belong in the first mobile release.
- [ ] Define an exchange-rate policy before any cross-currency comparison is introduced.
