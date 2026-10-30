# Backlog draft — Module 0 (Foundation)

Draft for review before creation on GitHub (see [process](../process.md)). Every issue below gets: Module = `Foundation`, the repository milestone in its section, the label and Size shown, and the listed blockers as issue dependencies. Keys such as `M0.1-01` are draft references only; GitHub assigns the issue numbers.

M0.5 (per-depot data visibility) is marked *later* in the ROADMAP and gets no issues yet.

**Status**: created on GitHub on 2026-10-30 as issues #1–#20, in key order (`M0.1-01` = #1 … `M0.4-04` = #20). GitHub is now the source of truth; this draft is kept as a record.

---

## M0.1 — Project structure, authentication, roles and permissions, user × depot link

### M0.1-01 · Choose the database and migration tool
`spike` · Size **S** · Blocked by: —

**Context**
The ROADMAP leaves the database open (*Pending technical decisions*). Nothing persistent can start without it.

**Question**
Which database and migration tool fit: immutable histories (readings, events, documents), "state on a date" queries, search by current or previous plate/fleet number, and local execution with Docker?

**Deliverable**
- [ ] ADR `docs/adr/0001-database.md` with the options considered, the decision and its consequences.
- [ ] No production code.

**Timebox**: half a day.

---

### M0.1-02 · Define the backend architecture and test strategy
`spike` · Size **S** · Blocked by: M0.1-01

**Context**
The ROADMAP requires business rules to live in the domain and be testable in isolation (*Principles*), and lists the testing strategy as a pending decision.

**Question**
How are modules laid out in `server/` (package per module, domain isolated from Spring and persistence through project-owned interfaces), which test levels exist (domain unit, persistence, API) and how the whole suite runs with a single command?

**Deliverable**
- [ ] ADR `docs/adr/0002-architecture-and-tests.md`.
- [ ] The single test command recorded in the README.

**Timebox**: half a day.

---

### M0.1-03 · Choose the authentication mechanism
`spike` · Size **XS** · Blocked by: M0.1-02

**Question**
Session cookie or stateless token (JWT) for the API, given the frontend is not chosen yet and the demo will be deployed publicly?

**Deliverable**
- [ ] ADR `docs/adr/0003-authentication.md`.

**Timebox**: 2 hours.

---

### M0.1-04 · Run the test suite on every push
`chore` · Size **S** · Blocked by: M0.1-02

**Context**
The Definition of Done requires the whole suite to pass with a single command; CI makes that visible on every pull request.

**Scope**
- GitHub Actions workflow running the single test command on push and pull request.
- Status badge in the README.

**Acceptance criteria**
- [ ] Given a pull request with a failing test, when CI runs, then the check fails and shows the failing test.
- [ ] Given a pull request with every test passing, when CI runs, then the check passes.

---

### M0.1-05 · Register users with a role
`feature` · Size **M** · Blocked by: M0.1-02

**Context**
Roles from ROADMAP Module 0: `admin`, `fleet_manager`, `maintenance`, `warehouse`, `planning`, `charter`, `read_only`. The e-mail is also the address for notifications (M0.4).

**Scope**
- Create, deactivate and list users: name, e-mail (unique), role, active flag.

**Out of scope**
Login (M0.1-06); depot link (M0.1-08).

**Acceptance criteria**
- [ ] Given a valid name, e-mail and role, when an admin creates a user, then the user is active with that role.
- [ ] Given an e-mail already in use, when creating a user, then the error shows the e-mail.
- [ ] Given a role outside the list, when creating a user, then the error lists the valid roles.
- [ ] Given a deactivated user, when listing active users, then that user is not listed.

**References**: ROADMAP Module 0 *Scope*.

---

### M0.1-06 · Authenticate users and protect the API
`feature` · Size **M** · Blocked by: M0.1-03, M0.1-05

**Scope**
- Login with e-mail and password (hashed), using the mechanism from ADR 0003.
- Every endpoint except login requires an authenticated, active user.

**Acceptance criteria**
- [ ] Given valid credentials of an active user, when logging in, then access is granted.
- [ ] Given a wrong password or an unknown e-mail, when logging in, then access is denied with the same message for both.
- [ ] Given a deactivated user, when logging in, then access is denied.
- [ ] Given no credentials, when calling any protected endpoint, then the response is 401.
- [ ] Passwords are never stored or logged in plain text.

---

### M0.1-07 · Enforce role permissions on endpoints
`feature` · Size **M** · Blocked by: M0.1-06

**Scope**
- A permission matrix (role × action) documented in `docs/permissions.md`, starting with the Foundation actions and extended by each module.
- Requests outside the role's permissions are rejected.

**Acceptance criteria**
- [ ] Given a `read_only` user, when calling a write endpoint, then the response is 403 and names the required permission.
- [ ] Given an `admin`, when calling any endpoint, then it is allowed.
- [ ] Given the matrix document, then every protected endpoint maps to one permission in it.

---

### M0.1-08 · Link users to depots
`feature` · Size **S** · Blocked by: M0.1-05, M0.2-01

**Context**
Notifications are sent per depot (ROADMAP Module 1 *Alerts*); later, data visibility may be restricted per depot (M0.5).

**Scope**
- Link a user to one or more depots; `admin` is implicitly linked to all.
- List the users of a depot by role.

**Acceptance criteria**
- [ ] Given a `fleet_manager` linked to depots A and B, when listing the `fleet_manager` users of depot B, then the user is listed.
- [ ] Given a non-admin user with no depot, when saving, then the error states at least one depot is required.
- [ ] Given a depot id that does not exist, when linking, then the error shows the id.

---

## M0.2 — Base records and system parameters

### M0.2-01 · Register the company and its depots
`feature` · Size **S** · Blocked by: M0.1-02

**Context**
One company, several depots, one state; every vehicle is registered in the headquarters municipality (ROADMAP Module 0 *Decisions*).

**Scope**
- Company: name, headquarters municipality.
- Depot: code (unique), name, municipality, active flag.

**Acceptance criteria**
- [ ] Given a depot code already in use, when creating a depot, then the error shows the code.
- [ ] Given a municipality outside the company's state, when creating a depot, then the error shows the municipality and the expected state.
- [ ] Given a deactivated depot, then it is not offered for new links but stays in history.

---

### M0.2-02 · Register municipalities
`feature` · Size **S** · Blocked by: M0.1-02

**Scope**
- Municipality: code (unique), name, state. Seeded with fictional municipalities of one state.

**Acceptance criteria**
- [ ] Given a code already in use, when creating a municipality, then the error shows the code.
- [ ] Given a list request, then municipalities are returned sorted by name.

---

### M0.2-03 · Register holidays by scope
`feature` · Size **S** · Blocked by: M0.2-02

**Context**
Module 3 resolves the day type (weekday, Saturday, Sunday/holiday) from the calendar and this registry.

**Scope**
- Holiday: date, name, scope (`national`, `state`, `municipal` with municipality).
- Query "is date D a holiday in municipality M".

**Acceptance criteria**
- [ ] Given a municipal holiday in M1, when asking about D in M2, then it is not a holiday.
- [ ] Given a national holiday, when asking about D in any municipality, then it is a holiday.
- [ ] Given a `municipal` holiday without a municipality, when saving, then the error states the municipality is required.

---

### M0.2-04 · Register cost centers
`feature` · Size **XS** · Blocked by: M0.1-02

**Scope**
- Cost center: code (unique), name, active flag.

**Acceptance criteria**
- [ ] Given a code already in use, when creating a cost center, then the error shows the code.

---

### M0.2-05 · Store typed system parameters with history
`feature` · Size **M** · Blocked by: M0.1-02

**Context**
Thresholds live in configuration, not code (ROADMAP *Principles*). Fleet alone needs: average window and minimum history, jump multiplier and ceiling, alert lead times, maximum age per service, useful life per class.

**Scope**
- Parameter: key, type (`integer`, `decimal`, `boolean`, `text`, `days`, `km`, `years`), description.
- Values with an effective-from date; reading a parameter returns the value in effect on a given date.
- Every change recorded with who and when.

**Acceptance criteria**
- [ ] Given a parameter of type `integer`, when saving `"abc"`, then the error shows the value and the expected type.
- [ ] Given value 90 from January and 60 from March, when reading on February 10, then the result is 90.
- [ ] Given a value change, then the previous value stays in the history.
- [ ] Given an unknown key, when reading, then the error shows the key.

---

## M0.3 — Audit trail and initial seed

### M0.3-01 · Record an audit trail for every change
`feature` · Size **M** · Blocked by: M0.1-06

**Context**
"Who changed what, when, previous/new value" (ROADMAP Module 0). The domain gets the acting user through a project-owned interface, so it does not depend on the authentication framework.

**Scope**
- Acting-user interface used by domain operations; a named fake implementation for tests.
- Audit entry: actor, timestamp, entity, entity id, field, previous value, new value.

**Acceptance criteria**
- [ ] Given a depot renamed from "North" to "North Depot", then an entry records the actor, the field `name` and both values.
- [ ] Given a creation, then an entry records every initial field with an empty previous value.
- [ ] Audit entries cannot be changed or deleted through the API.

---

### M0.3-02 · Query the audit trail of a record
`feature` · Size **S** · Blocked by: M0.3-01

**Acceptance criteria**
- [ ] Given a record with three changes, when querying its trail, then the three entries come newest first.
- [ ] Given a user without the audit permission, when querying, then the response is 403.

---

### M0.3-03 · Generate the foundation seed
`feature` · Size **M** · Blocked by: M0.2-01, M0.2-02, M0.2-03, M0.2-05, M0.1-05

**Context**
Fictional data realistic enough to demonstrate every flow (ROADMAP *Principles*). Module seeds (e.g. the fleet) build on this one.

**Scope**
- Company, 3 depots, fictional municipalities of one state, holidays for the current and next year, one user per role, initial system parameters.
- Deterministic (fixed random seed) and idempotent; runs only in development/demo profiles.

**Acceptance criteria**
- [ ] Given an empty database, when the seed runs twice, then the second run creates nothing new.
- [ ] Given two runs on empty databases, then both produce the same data.
- [ ] Given the production profile, then the seed does not run.

---

## M0.4 — Notifications and file storage

### M0.4-01 · Send e-mail through a project-owned interface
`feature` · Size **M** · Blocked by: M0.1-02

**Scope**
- Project-owned e-mail interface; SMTP adapter (Spring Mail); named fake for tests.
- Mailpit in the local Docker setup to inspect sent e-mails.

**Acceptance criteria**
- [ ] Given a message to a valid address, when sent locally, then it appears in Mailpit.
- [ ] Given an SMTP failure, then the error is logged as structured JSON with the recipient and the reason, and the caller is told the send failed.
- [ ] Domain code depends only on the project interface.

---

### M0.4-02 · Log sent notifications to avoid repeats
`feature` · Size **S** · Blocked by: M0.4-01

**Context**
The daily digest must not repeat the same alert every day; it is resent only when an item changes from "due soon" to "expired" (ROADMAP Module 1 *Alerts*).

**Scope**
- Sent-notification log: recipient, subject reference (e.g. document id), state notified, timestamp.
- Query "was state S of reference R already notified to recipient X".

**Acceptance criteria**
- [ ] Given "due soon" already notified for document D, when checking "due soon" for D, then it reports already sent.
- [ ] Given "due soon" notified for D, when checking "expired" for D, then it reports not sent.

---

### M0.4-03 · Unsubscribe from notifications
`feature` · Size **S** · Blocked by: M0.4-01, M0.1-05

**Acceptance criteria**
- [ ] Given a user who unsubscribed from a notification kind, when that notification is sent, then the user is skipped.
- [ ] Every notification e-mail includes the unsubscribe link for its kind.
- [ ] Given an invalid or expired unsubscribe link, then the error states the link is invalid and nothing changes.

---

### M0.4-04 · Store attachments through a project-owned interface
`feature` · Size **M** · Blocked by: M0.1-02

**Context**
Fleet documents carry attachments (CRLV PDF, insurance policy, certificates).

**Scope**
- Project-owned storage interface; local-disk adapter (development) and S3-compatible adapter (MinIO in the demo environment); named fake for tests.
- Store, download and list by owner; configurable maximum size and allowed types (PDF, JPEG, PNG).

**Acceptance criteria**
- [ ] Given a 2 MB PDF, when stored and downloaded, then the bytes are identical.
- [ ] Given a file above the size limit, then the error shows the file size and the limit.
- [ ] Given a `.exe` file, then the error shows the type and the allowed types.
- [ ] Switching between disk and MinIO is configuration only.
