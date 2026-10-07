
# Archer Feature Specification

## Purpose

Archer is a freelance marketplace where clients publish work, freelancers discover opportunities and submit proposals, and both parties manage agreed work through completion.

This document defines **what Archer should do**. It is intentionally independent of framework, database, repository, and implementation decisions. Those belong in project documentation and `AGENTS.md`.

## Product Principles

- Make good projects and capable people easier to find.
- Keep project expectations, budgets, status, and ownership clear.
- Support both client and freelancer workflows without payment processing in the initial release.
- Treat trust, profile quality, reviews, and predictable lifecycle transitions as core product behavior.
- Show USD and MMK explicitly without implying conversion.
- Make the web interface available in English and Burmese.
- Support accessible light and dark appearance modes.

## Users And Roles

### Client

A client can create and manage projects, review proposals, select a freelancer, manage project status, and review completed work.

### Freelancer

A freelancer can maintain a public profile, discover projects, submit proposals, manage accepted work, and review completed work.

### Admin

An admin can moderate users and marketplace content and inspect platform activity.

A person may act as both a client and freelancer unless a later product decision restricts this.

## Initial Release Scope

### Included

- Account registration, login, logout, session refresh, profile management, password change, and password reset.
- Public project discovery with search, filters, sorting, pagination, and project details.
- Freelancer profiles with skills, portfolio items, availability, rates, ratings, and completed project count.
- Client project creation, editing, publishing, closing, and deletion.
- Freelancer proposals with price, currency, cover letter, duration, editing, withdrawal, and status tracking.
- Proposal review and selection by the project client.
- Active project status and completion/cancellation workflow.
- One review per party for a completed project.
- In-app notifications for important marketplace events.
- Development data sufficient for demos and UI testing.

### Not Included Initially

- Payment processing, escrow, invoices, withdrawals, or transaction reconciliation.
- Real-time chat, video calls, time tracking, or automated work logs.
- Native push notification delivery.
- Organizations, teams, or multi-tenant accounts.
- Advanced dispute resolution.

## Feature Requirements

### 1. Accounts And Authentication

Users can:

- Register with name, email, and password.
- Log in and log out.
- Restore a session after an access token expires.
- View and update their profile.
- Change their password.
- Request and complete a password reset.

The product must:

- Enforce role-aware permissions.
- Never expose passwords, password hashes, token secrets, or refresh-token values.
- Show clear loading, validation, error, and unauthorized states.

### 2. Freelancer Profiles

A public freelancer profile may include:

- Display name, headline, biography, location, and avatar.
- Skills and optional experience/proficiency details.
- Hourly rate and currency.
- Availability.
- Portfolio items with title, description, URL, and image.
- Average rating, review count, and completed project count.

Users can browse public freelancer profiles without seeing private account data.

### 3. Project Discovery

Users can:

- Browse published projects.
- Search by project title and description.
- Filter by category, skill, currency, budget range, status, and date where available.
- Sort by newest, deadline, budget, or relevance where available.
- Paginate results.
- Open a project detail page.
- See the project owner, category, skills, budget range, currency, deadline, proposal count, and current public status.

Every monetary value displays its currency. USD and MMK are never silently converted.

### 4. Client Project Management

A client can:

- Create a draft project.
- Set title, description, category, skills, budget range, currency, and deadline.
- Edit or delete a project they own.
- Publish a draft.
- Review proposals for their project.
- Close a project that is no longer accepting work.
- Accept one proposal and move the project into active work.
- Mark active work completed.
- Cancel work with a recorded reason.

Project statuses:

`DRAFT`, `PUBLISHED`, `IN_REVIEW`, `IN_PROGRESS`, `COMPLETED`, `CANCELLED`, `CLOSED`

Only the owning client or an authorized admin can perform owner actions.

### 5. Proposals

A freelancer can:

- Submit one proposal per project while proposals are accepted.
- Provide a cover letter, proposed price, currency, estimated duration, and optional links or attachments.
- Edit or withdraw their proposal before selection.
- View proposal status.

A client can:

- View and compare proposals for their projects.
- Shortlist, reject, archive, or accept proposals.
- Accept no more than one proposal for a project.

Proposal statuses:

`SUBMITTED`, `SHORTLISTED`, `ACCEPTED`, `REJECTED`, `WITHDRAWN`

When a proposal is accepted:

1. It becomes `ACCEPTED`.
2. The project becomes `IN_PROGRESS`.
3. Other active proposals become `REJECTED`.
4. The client and selected freelancer can view the active project summary.
5. No payment or escrow record is created.

### 6. Project Completion And Cancellation

The client can mark an active project `COMPLETED`. After completion, reviews become available.

A permitted cancellation records:

- The resulting `CANCELLED` status.
- The person who cancelled.
- The cancellation reason.
- The relevant timestamp.

Invalid lifecycle transitions must be rejected clearly.

### 7. Reviews

After a project is completed:

- The client may review the freelancer.
- The freelancer may review the client.
- Each party can submit at most one review for the project.
- A review contains a rating from 1 to 5, an optional comment, and a creation timestamp.
- Reviews are immutable after submission in the initial release.
- Aggregate ratings use valid reviews from completed projects.

### 8. Notifications

The product creates in-app notifications for:

- Proposal submitted.
- Proposal accepted or rejected.
- Project status changed.
- Review received.
- Password reset requested.

Users can view notifications and mark them read. Email, SMS, and push delivery are outside the initial scope.

### 9. Currency

The initial product supports exactly:

- `USD`
- `MMK`

Rules:

- Every budget, rate, and proposal price has an explicit currency.
- Unsupported currencies are rejected.
- No exchange-rate conversion is shown.
- No comparison should imply that an amount in USD equals an amount in MMK.
- Amounts must retain precision appropriate for money.

### 10. Language

The web interface supports:

- English (`en`)
- Burmese (`my`)

Users can switch language from public and authentication screens. The selected language persists on the same device. Interface copy, accessibility labels, validation feedback, and relative dates use the selected language. User-authored and server-authored marketplace content is displayed in its original language and is not translated automatically.

### 11. Appearance

The web interface supports light and dark modes. On a user's first visit, the interface follows the operating-system color preference. Users can switch modes from public and authentication screens, and their explicit selection persists on the same device. Both modes preserve readable contrast, visible focus states, and the established English and Burmese typography behavior.

## Acceptance Criteria

The initial feature release is acceptable when:

- A user can register, log in, restore a session, and log out.
- A user can browse, search, filter, paginate, and open published projects.
- A client can create, publish, edit, close, and manage an owned project.
- A freelancer can submit, edit, withdraw, and track a proposal.
- A client can accept one proposal and the project transitions correctly.
- A completed project enables one review from each party.
- Notifications are created for the defined marketplace events.
- USD and MMK are validated and displayed explicitly throughout the product.
- A user can switch the web interface between English and Burmese, and the choice persists after reload.
- A user can switch between light and dark modes, and the choice persists after reload.
- Unauthorized users cannot access private resources or mutate another user's resources.
- No payment functionality is exposed.

## Feature Status

Implemented or partially implemented:

- API health and authentication foundation.
- Prisma data model and deterministic development seed data.
- Public project discovery API with pagination, search, category, skill, currency, and budget filters.
- Public project detail API.
- Web project discovery and detail screens connected to the API.
- Web login flow connected to the API.
- English and Burmese web interface support with a persistent language switch.
- Light and dark web appearance with a persistent theme switch.

Next feature slices:

1. Client project creation and publishing.
2. Proposal submission and management.
3. Proposal acceptance and project lifecycle actions.
4. Freelancer and client profile screens.
5. Reviews and notifications.
6. Mobile client implementation.

## Open Product Decisions

These decisions can change later without changing the feature structure above:

1. Can users switch between client and freelancer modes, or is a role fixed at registration?
2. Are budgets fixed-price, hourly, or both?
3. Are file attachments required in the first release, and what storage service will hold them?
4. Is public browsing available without authentication?
5. Which languages and regions are supported beyond English and Burmese?
6. Should admin moderation screens ship in the first release?
7. Should push notifications be added to the first mobile release?
8. What future exchange-rate policy should apply if cross-currency comparisons are requested?
