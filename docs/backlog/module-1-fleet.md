# Backlog draft — Module 1 (Fleet)

Draft for review before creation on GitHub (see [process](../process.md)). Every issue below gets: Module = `Fleet`, the repository milestone in its section, the label and Size shown, and the listed blockers as issue dependencies. Keys such as `M1.1-01` are draft references only; `M0.x-yy` keys point to [module-0-foundation.md](module-0-foundation.md).

Rules come from [ROADMAP.md](../../ROADMAP.md) Module 1; section names are cited as *§ Name*. Every write operation records an audit entry (M0.3-01); that is not repeated per issue.

M1.8 (hour meter and onboard equipment) is marked *later* and gets no issues yet.

**Status**: created on GitHub on 2026-10-30 as issues #21–#52, in key order (`M1.1-01` = #21 … `M1.7-02` = #52; the parent `M1.4-00` = #39). GitHub is now the source of truth; this draft is kept as a record.

---

## M1.1 — Vehicle registry

### M1.1-01 · Register a vehicle with internal id and VIN validation
`feature` · Size **M** · Blocked by: M0.2-01, M0.3-01

**Context**
First slice of the registry. Identity is the internal id; the fleet number may change (*§ Identification*).

**Scope**
- Register: fleet number, VIN, chassis make/model, manufacturing and model year, class, seated and standing capacity, depot, service entry date and km.
- Read a vehicle by id.

**Out of scope**
Plate, RENAVAM, accessibility, optional characteristics and fleet number history: each has its own issue.

**Acceptance criteria**
- [ ] Given a 17-character alphanumeric VIN, when registering, then the vehicle gets an internal id and status `in_service`.
- [ ] Given VIN `9BW123`, then the error shows the value and that 17 alphanumeric characters are expected.
- [ ] Given a VIN already registered, then the duplicate error shows the VIN.
- [ ] Given fleet number `1042` in use by a non-retired vehicle, when registering another with `1042`, then the duplicate error shows the fleet number.
- [ ] Given a class outside `micro` … `coach`, then the error lists the valid classes.
- [ ] Given a negative service entry km, then the error shows the value.
- [ ] Given a model year more than one year after the manufacturing year, then the error shows both years.

**References**: *§ Entities*, *§ Identification*; CONTRAN Res. 968/2022.

---

### M1.1-02 · Validate plate format and uniqueness
`feature` · Size **S** · Blocked by: M1.1-01

**Scope**
- Plate on registration, accepting the old (`AAA9999`) and Mercosul (`AAA9A99`) formats; stored uppercase without hyphen.

**Acceptance criteria**
- [ ] Given `abc-1234`, when saving, then the plate is stored as `ABC1234`.
- [ ] Given `ABC1C34`, then it is accepted as Mercosul.
- [ ] Given `AB12345`, then the error shows the value and both expected formats.
- [ ] Given a plate that is the current plate of another vehicle, then the duplicate error shows the plate and that vehicle's fleet number.

**References**: *§ Identification*; CONTRAN Res. 969/2022.

---

### M1.1-03 · Change plate with history and search by previous plate
`feature` · Size **M** · Blocked by: M1.1-02

**Context**
Plates change by law (e.g. conversion to Mercosul); history must not be lost and the old plate must still find the vehicle.

**Scope**
- Plate change event: previous plate, new plate, date, reason.
- Find a vehicle by current or any previous plate.

**Out of scope**
Automatic old-to-Mercosul conversion.

**Acceptance criteria**
- [ ] Given a vehicle with `ABC1234`, when changing to `ABC1C34`, then the current plate is `ABC1C34` and the history keeps `ABC1234` with date and reason.
- [ ] Given that change, when searching `ABC1234` or `ABC1C34`, then the same vehicle is found.
- [ ] Given `XYZ9876` is a current or previous plate of another vehicle, when changing to it, then the error shows the plate and the other vehicle's fleet number.

**References**: *§ Identification*; acceptance criterion "found by searching either plate".

---

### M1.1-04 · Change fleet number with history
`feature` · Size **S** · Blocked by: M1.1-01

**Scope**
- Fleet number change event: previous, new, date, reason. No format enforced.
- Find a vehicle by current or any previous fleet number.

**Out of scope**
Releasing the number on retirement (M1.3-03).

**Acceptance criteria**
- [ ] Given vehicle `1042`, when changing to `2042`, then the current number is `2042` and the history keeps `1042`.
- [ ] Given `2042` in use by another non-retired vehicle, when changing to it, then the duplicate error shows the number.
- [ ] Given that change, when searching `1042`, then the vehicle is found and shown as `2042`.

---

### M1.1-05 · Validate RENAVAM with a check-digit warning
`feature` · Size **S** · Blocked by: M1.1-01

**Context**
The check-digit algorithm was not confirmed in a primary source, so a wrong digit only warns (*§ Identification*).

**Acceptance criteria**
- [ ] Given a 9-digit RENAVAM, when saving, then it is stored left-padded to 11 digits.
- [ ] Given a value with letters or more than 11 digits, then the error shows the value and that up to 11 digits are expected.
- [ ] Given 11 digits with a wrong check digit, then the vehicle is saved and the response carries a warning with the value.
- [ ] Given a RENAVAM already registered, then the duplicate error shows the value.

**References**: research [24], [25].

---

### M1.1-06 · Derive accessibility from structured fields
`feature` · Size **S** · Blocked by: M1.1-01

**Scope**
- Fields: access type (`a`, `b`, `c` or none), wheelchair spaces, priority seats.
- Derived `accessible = access_type ∈ {a, b, c} ∧ wheelchair_spaces ≥ 1`; never typed.

**Acceptance criteria**
- [ ] Given access type `a` and 1 wheelchair space, then the vehicle is accessible.
- [ ] Given access type `c` and 0 wheelchair spaces, then it is not accessible.
- [ ] Given 40 seats and 3 priority seats, then it is saved with a warning that at least 4 are expected.
- [ ] A request trying to set `accessible` directly is rejected.

**References**: *§ Accessibility*; ABNT NBR 14022.

---

### M1.1-07 · Record body and equipment characteristics
`feature` · Size **XS** · Blocked by: M1.1-01

**Scope**
- Body make/model, body number (NICV, optional), fuel, air conditioning (optional), number of doors (optional).

**Acceptance criteria**
- [ ] Given a NICV that is not 21 characters, then the error shows the value and the expected length.
- [ ] Given a fuel outside the configured list, then the error lists the valid fuels.

---

### M1.1-08 · List and search vehicles with filters
`feature` · Size **M** · Blocked by: M1.1-03, M1.1-04, M1.1-06

**Scope**
- Paginated list with filters: status, class, depot, accessible, age range.
- Search by current or previous fleet number, current or previous plate, VIN, RENAVAM.
- Age is counted from the manufacturing year.

**Out of scope**
Document and eligibility filters (M1.4-07).

**Acceptance criteria**
- [ ] Given vehicles in three depots, when filtering by depot B, then only depot B vehicles are returned.
- [ ] Given a vehicle manufactured in 2018 with model year 2019, when filtering age ≥ 8 in 2026, then it is returned.
- [ ] Given a page size above the maximum, then the error shows the value and the maximum.

---

### M1.1-09 · Generate the fictional fleet in the seed
`feature` · Size **S** · Blocked by: M0.3-03, M1.1-08

**Scope**
- About 120 vehicles across 3 depots, every class represented, a mix of old and Mercosul plates, some accessible and some not, a few plate and fleet number changes.

**Acceptance criteria**
- [ ] Every seeded vehicle passes the registry validations.
- [ ] Every class and every depot has at least one vehicle.
- [ ] The seed stays deterministic and idempotent.

---

## M1.2 — Odometer

### M1.2-01 · Record monotonic odometer readings
`feature` · Size **M** · Blocked by: M1.1-01

**Context**
Odometer is the km source for Maintenance and Metrics (*§ Odometer*). Manual entry only; `fueling` and `telematics` sources are reserved.

**Scope**
- Reading: vehicle, date/time, km, source, recorded by (acting user).
- Readings are never edited.

**Acceptance criteria**
- [ ] Given a last reading of 121,500 km, when recording 120,000 km, then the error shows both values.
- [ ] Given a reading dated before the service entry date, then the error shows both dates.
- [ ] Given a future date, then the error shows the date.
- [ ] Given a source other than `manual`, then the error states only manual entry is enabled.
- [ ] No endpoint edits or deletes a reading.

---

### M1.2-02 · Replace the odometer keeping accumulated km
`feature` · Size **M** · Blocked by: M1.2-01

**Scope**
- Replacement event: date, final km of the old unit, initial km of the new one.
- Accumulated km stays continuous across replacements.

**Acceptance criteria**
- [ ] Given accumulated 300,000 km, a replacement with old final 300,000 and new initial 0, and a later reading of 1,000, then accumulated km is 301,000.
- [ ] Given that replacement, when recording 500 km on the new unit, then it is accepted.
- [ ] Given an old final km lower than the last reading, then the error shows both values.

---

### M1.2-03 · Void a reading with justification
`feature` · Size **S** · Blocked by: M1.2-01

**Acceptance criteria**
- [ ] Given a typo reading, when voided with a justification, then it stays in history marked voided with who, when and why.
- [ ] Given a voided reading, then it is ignored by the monotonic check and by the average.
- [ ] Given an empty justification, then the error states it is required.

---

### M1.2-04 · Compute the average km per day
`feature` · Size **M** · Blocked by: M1.2-02, M0.2-05

**Context**
Used by Module 2 only to project due dates (*§ Average km/day*).

**Scope**
- `(accumulated_km(today) − accumulated_km(today − window)) / window_days`, maintenance days included.
- Parameters: window (90 days), minimum history days (14) and km (500).
- Below the minimum, the average of the same class, flagged `estimated`.

**Acceptance criteria**
- [ ] Given 18,000 km accumulated over the last 90 days, then the average is 200 km/day, not estimated.
- [ ] Given a vehicle with readings covering 10 days, then the class average is returned flagged `estimated`.
- [ ] Given an odometer replacement inside the window, then the average uses accumulated km and is not distorted.
- [ ] Given the window parameter changed to 60, then the next calculation uses 60 days.

---

### M1.2-05 · Alert on anomalous odometer jumps
`feature` · Size **S** · Blocked by: M1.2-04

**Scope**
- A reading implying a daily average above N × the vehicle average (3×) or above an absolute ceiling (800 km/day) needs a justification to be saved.

**Acceptance criteria**
- [ ] Given an average of 200 km/day and a reading implying 700 km/day, then saving without justification fails and the error shows both rates and the 3× threshold.
- [ ] Given the same reading with a justification, then it is saved with the justification.
- [ ] Given a reading implying 900 km/day for any vehicle, then the ceiling rule applies.

---

## M1.3 — Lifecycle

### M1.3-01 · Change vehicle status through the state machine
`feature` · Size **M** · Blocked by: M1.1-01

**Scope**
- Transitions: `in_service` ⇄ `reserve`; `in_service`/`reserve` → `in_maintenance` → `in_service`/`reserve`.
- Manual changes require a justification; each change records a status event.

**Out of scope**
Work-order driven changes (Module 2); retirement (M1.3-03).

**Acceptance criteria**
- [ ] Given `reserve`, when changing to `in_service` with a justification, then the event records previous, new, date, reason.
- [ ] Given `in_maintenance`, when changing to `in_maintenance`, then the error shows the current status and the allowed targets.
- [ ] Given no justification, then the error states it is required.

---

### M1.3-02 · Query vehicle status on a date and its timeline
`feature` · Size **S** · Blocked by: M1.3-01

**Acceptance criteria**
- [ ] Given `in_service` until March 10 and `in_maintenance` after, when asking for March 5 and March 12, then the answers are `in_service` and `in_maintenance`.
- [ ] Given a date before the service entry date, then the error shows both dates.
- [ ] The timeline lists status events newest first.

---

### M1.3-03 · Retire a vehicle with a reason
`feature` · Size **M** · Blocked by: M1.3-01, M1.2-01, M1.1-04

**Scope**
- Reason (`sale`, `total_loss`, `end_of_life`, `dismantling`), date, final km, optional sale value.
- Informational reminders: `sale` → notify DETRAN (CTB art. 134); `dismantling` → request deregistration (CTB art. 126).
- A retired vehicle accepts no readings, plate or fleet number changes, or transfers; its fleet number is released.

**Acceptance criteria**
- [ ] Given a final km lower than the last reading, then the error shows both values.
- [ ] Given a retired vehicle, when recording a reading, then the error states the vehicle is retired and since when.
- [ ] Given retired vehicle `1042`, when registering a new vehicle with `1042`, then it is accepted.
- [ ] Given reason `sale`, then the response includes the DETRAN reminder.
- [ ] Given `retired`, when changing to any status, then the error states retirement is terminal.

---

### M1.3-04 · Transfer a vehicle between depots
`feature` · Size **S** · Blocked by: M1.1-01

**Scope**
- Transfer event: from, to, effective date, reason. Depot on a date comes from the transfer history.
- Never changes plate or registration municipality.

**Acceptance criteria**
- [ ] Given a transfer from A to B effective April 1, when asking the depot on March 31 and April 1, then the answers are A and B.
- [ ] Given a transfer to the current depot, then the error shows the depot.
- [ ] Given an effective date before the last transfer, then the error shows both dates.

---

## M1.4 — Documents and eligibility

### M1.4-00 · Vehicle eligibility per service *(parent)*
`feature` · Size **L** (split) · Sub-issues: M1.4-01 … M1.4-06

**Context**
Scheduling (M3.4) only accepts vehicles eligible for the route's service. Eligibility is derived: every `blocking` document required by the service is current **and** the vehicle is within the service's maximum age (*§ Documents and eligibility*).

**Acceptance criteria** (checked when every sub-issue is done)
- [ ] Given an expired school transport authorization and current urban documents, then the vehicle is not eligible for `school` and is eligible for `urban`.
- [ ] Given a vehicle manufactured in 2015, when asking eligibility for `urban` in 2026, then it is not eligible and the reason cites the 10-year maximum age.
- [ ] Given a non-eligible vehicle, then the answer lists **every** reason, not only the first.

---

### M1.4-01 · Configure document types
`feature` · Size **S** · Parent: M1.4-00 · Blocked by: M0.2-05

**Scope**
- Type: name, required services, validity rule (fixed months, entered date, or derived from plate), severity (`blocking`/`warning`), alert lead time (30 days).
- Seeded with the seven initial types of *§ Documents and eligibility*, all `blocking`.

**Acceptance criteria**
- [ ] Given a type with no required service, then the error states at least one is required.
- [ ] Given a service outside the four, then the error lists the valid services.

---

### M1.4-02 · Record documents and renewals without overwriting
`feature` · Size **M** · Parent: M1.4-00 · Blocked by: M1.4-01, M1.1-01

**Acceptance criteria**
- [ ] Given a semiannual school inspection issued January 10, then it expires July 10.
- [ ] Given a renewal of a type, then the previous document stays in history and the new one is current.
- [ ] Given an expiry date before the issue date, then the error shows both dates.
- [ ] Given the current document of a type on a date, then it is the latest one valid on that date.

---

### M1.4-03 · Attach files to documents
`feature` · Size **S** · Parent: M1.4-00 · Blocked by: M1.4-02, M0.4-04

**Acceptance criteria**
- [ ] Given a document, when attaching a PDF, then it is listed and downloadable from the document.
- [ ] Given a renewal, then attachments of the previous document stay with it.

---

### M1.4-04 · Derive the licensing due date from the plate
`feature` · Size **S** · Parent: M1.4-00 · Blocked by: M1.4-02, M1.1-02

**Scope**
- Configurable table "plate last digit → deadline month" (initial: 1–2 Sep, 3–5 Oct, 6–8 Nov, 9–0 Dec).
- The user records that the year's licensing was paid; the due date is never typed.

**Acceptance criteria**
- [ ] Given plate `ABC1C34` (last digit 4), then the CRLV due date for the year is the end of October.
- [ ] Given a plate change that changes the last digit, then the next due date follows the new digit.
- [ ] Given an attempt to type the CRLV expiry date, then it is rejected.

---

### M1.4-05 · Configure maximum age per service
`feature` · Size **XS** · Parent: M1.4-00 · Blocked by: M0.2-05

**Acceptance criteria**
- [ ] Given parameters `urban` 10, `intercity` 15, `charter` 15, `school` 15, then each service reads its own value.
- [ ] Age is counted from the manufacturing year, never the model year.

---

### M1.4-06 · Query vehicle eligibility for a service on a date
`feature` · Size **M** · Parent: M1.4-00 · Blocked by: M1.4-02, M1.4-04, M1.4-05

**Acceptance criteria**
- [ ] The parent's three acceptance criteria.
- [ ] Given a `warning` document expired, then the vehicle stays eligible and the answer lists the warning.
- [ ] Given a retired vehicle, then it is eligible for no service.

---

### M1.4-07 · Filter vehicles by document situation and eligible service
`feature` · Size **S** · Blocked by: M1.4-06, M1.1-08

**Acceptance criteria**
- [ ] Given a filter "eligible for `school`", then only eligible vehicles are returned.
- [ ] Given a filter "documents expired", then vehicles with at least one expired current document are returned.

---

## M1.5 — Alerts

### M1.5-01 · Show the expiry panel per depot
`feature` · Size **M** · Blocked by: M1.4-02, M1.4-05

**Scope**
- Documents "due soon" (within the type's lead time) and "expired", and vehicles approaching the maximum age (12 months), filterable by depot.

**Acceptance criteria**
- [ ] Given a lead time of 30 days and a document expiring in 20 days, then it is listed as due soon.
- [ ] Given a vehicle that reaches the `urban` maximum age in 8 months, then it is listed as approaching.
- [ ] Given a depot filter, then only that depot's vehicles appear.

---

### M1.5-02 · Send the daily expiry digest per depot
`feature` · Size **M** · Blocked by: M1.5-01, M0.4-02, M0.4-03, M0.1-08

**Scope**
- Scheduled job at a configurable time (07:00) sending one e-mail per depot to its `fleet_manager` users.
- Items already notified in the same state are skipped; resent when "due soon" becomes "expired".

**Acceptance criteria**
- [ ] Given two depots with due items, then each depot's managers receive only their depot's items.
- [ ] Given a document notified as due soon yesterday and still due soon, then today's digest omits it.
- [ ] Given that document expires, then the next digest includes it as expired.
- [ ] Given a depot with nothing new, then no e-mail is sent.

---

## M1.6 — Vehicle profile and fleet panel

### M1.6-01 · Show the vehicle profile
`feature` · Size **M** · Blocked by: M1.3-02, M1.3-04, M1.4-06, M1.2-04

**Scope**
- Data, status timeline, fleet number/plate/depot history, km series, current documents, eligibility per service.

**Out of scope**
Linked work orders (Module 2).

**Acceptance criteria**
- [ ] Given a vehicle with a plate change and a transfer, then both appear in the profile history with dates.
- [ ] Given a vehicle eligible for `urban` only, then the profile shows each service with its reasons.

---

### M1.6-02 · Show the fleet panel indicators
`feature` · Size **M** · Blocked by: M1.4-06, M1.1-08

**Scope**
- Count by status, average age, % accessible, % above useful life, documents due/expired, eligible vehicles per service; filterable by depot.
- Useful life per class parameter (`micro`/`mini` 10, others 14).

**Acceptance criteria**
- [ ] Given 10 vehicles of which 3 are accessible, then % accessible is 30%.
- [ ] Given a `mini` manufactured 11 years ago, then it counts as above useful life; a `padron` of the same age does not.
- [ ] Retired vehicles are excluded from every indicator.

---

## M1.7 — Bulk import

### M1.7-01 · Import vehicles from CSV or XLSX
`feature` · Size **M** · Blocked by: M1.1-08

**Scope**
- Every line validated with the registry rules; errors reported per line (line, field, value, rule).
- Imports only if there are no errors (all or nothing); warnings (e.g. RENAVAM check digit) require confirmation for the batch.

**Acceptance criteria**
- [ ] Given 50 vehicles and one invalid VIN, then nothing is imported and the report shows line, field, value and rule.
- [ ] Given a file with a RENAVAM warning, when imported without confirmation, then nothing is imported and the warning is listed.
- [ ] Given the same file as CSV and as XLSX, then the results are identical.

---

### M1.7-02 · Import odometer readings from CSV or XLSX
`feature` · Size **M** · Blocked by: M1.7-01, M1.2-05

**Acceptance criteria**
- [ ] Given a reading lower than the vehicle's last one, then nothing is imported and the line is reported.
- [ ] Given two readings of the same vehicle in one file, then they are validated in date order.
- [ ] Given an anomalous jump, then the batch requires confirmation with a justification.
