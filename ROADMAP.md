# FrotaOps — Roadmap

Fleet management and transport operations platform for a **fictional** bus company. All data (vehicles, drivers, routes, municipalities, contracts) is generated for demonstration purposes. No real data is used.

## Overview

The system covers four operational pillars, supported by a shared foundation and a cross-cutting metrics layer:

| # | Module | Question it answers |
|---|--------|---------------------|
| 0 | Foundation | Who can do what, and what happened? |
| 1 | Fleet | Which vehicles do we have and what state are they in? |
| 2 | Maintenance | What needs to be done, how much did it cost, and how reliable is the fleet? |
| 3 | Operations Planning | Which vehicle and which driver run each trip? |
| 4 | Charter | What was contracted, what was delivered, and how much to bill? |
| 5 | Metrics | How is the operation performing? |

### Module dependencies

```
Foundation ──► Fleet ──► Maintenance ──┐
                 │                     ├──► Metrics
                 └──► Planning ────────┤
                         │             │
                         └──► Charter ─┘
```

- **Maintenance** depends on Fleet (vehicle, odometer, status).
- **Planning** depends on Fleet (availability, service eligibility) and indirectly on Maintenance (a vehicle with an open work order cannot be scheduled).
- **Charter** depends on Planning (completed trips are the basis for measurement).
- **Metrics** consumes data from every module.

### Principles

- **Business rules live in the domain**, not in the UI: validations (preventive maintenance due dates, scheduling conflicts, SLA deductions) must be testable in isolation.
- **Immutable history**: odometer readings, stock movements and closed measurements are never edited, only corrected through reversal/adjustment entries.
- **Configurable parameters**: thresholds such as delay tolerance, minimum rest period and preventive alert window live in configuration, not hardcoded.
- **Fictional seed data** realistic enough to demonstrate every flow.

---

## Module 0 — Foundation

### Goal
Provide the technical and cross-cutting base on which the business modules are built.

### Scope
- Authentication and session
- Access roles (RBAC): `admin`, `fleet_manager`, `maintenance`, `warehouse`, `planning`, `charter`, `read_only`
- User × depot link: each user belongs to one or more depots (`admin` sees all); used to route notifications per depot
- Base records: company/depots, cost centers, municipalities, holidays
- Audit trail (who changed what, when, previous/new value)
- System parameters (typed key/value, with history)
- **Notifications**: e-mail sent through a project-owned interface (SMTP via Spring Mail; Mailpit in development), scheduled jobs, and a sent-notification log so the same alert is not repeated every day; users can unsubscribe
- **File storage**: attachments stored through a project-owned interface (local disk in development, S3-compatible MinIO in the demo environment)
- Fictional data generator (seed)

### Decisions
- **Single company, multiple depots from day one** (seed: 3 depots), all in **one state**. Every vehicle is registered in the municipality of the company headquarters, regardless of depot.
- Per-depot data visibility (each user sees only their depots) is **later**; at first every user sees every depot and filters by depot. The user × depot link already prepares for it.

### Milestones
- **M0.1** — Project structure, authentication, roles and permissions, user × depot link
- **M0.2** — Base records and system parameters
- **M0.3** — Audit trail and initial seed
- **M0.4** — Notifications (e-mail, scheduled jobs, sent log) and file storage
- **M0.5** (later) — Per-depot data visibility

### Open questions
- Stack (frontend, database) and deployment strategy

---

## Module 1 — Fleet

### Goal
Be the source of truth for every vehicle: identification, characteristics, mileage, operational status, documents and **service eligibility** (which services each vehicle may run).

Supporting research, with sources: [docs/research/fleet-module.md](docs/research/fleet-module.md).

### Operating context (assumptions)
- The fictional company runs four **services**: `urban` (municipal), `intercity` (within the state, regulated by a fictional state agency), `charter` (contracts with municipalities) and `school` (school charter, with the extra requirements of the Brazilian Traffic Code, CTB arts. 136–139). Interstate charter (ANTT) is out of scope.
- The fleet is **company-owned** (no leasing) and registered in the **"aluguel"** DETRAN category (paid passenger transport, CTB arts. 96 and 135); neither is stored as a field.
- One state, multiple depots, every vehicle registered in the headquarters municipality (see Module 0).

### Entities
- **Vehicle**: internal id (identity), fleet number, license plate, chassis/VIN (unique, immutable), RENAVAM (unique, immutable), chassis make/model, body make/model, body number (NICV, optional), manufacturing/model year, **class** (`micro`, `mini`, `midi`, `basic`, `padron`, `articulated`, `coach`), capacity (seated/standing, as on the CRLV), accessibility (see below), air conditioning (optional), number of doors (optional), fuel, service entry date and km, current depot
- **Accessibility** (ABNT NBR 14022): access type (`a` low floor, `b` high floor with boarding platform, `c` high floor with lift), number of wheelchair spaces, number of priority seats; **accessible** is derived, never typed
- **Fleet number change** and **plate change**: vehicle, previous value, new value, date, reason
- **Depot transfer**: vehicle, from depot, to depot, effective date, reason
- **Odometer reading**: vehicle, date/time, km, source (manual, fueling, work order, telematics), recorded by, voided (with justification)
- **Odometer replacement**: vehicle, date, final km of the old unit, initial km of the new one
- **Status event**: vehicle, previous status, new status, date/time, reason, reference (e.g. work order)
- **Retirement**: reason (`sale`, `total_loss`, `end_of_life`, `dismantling`), date, final km, sale value (optional)
- **Document type** (configurable): name, services that require it, validity rule, severity (`blocking` | `warning`), alert lead time
- **Document**: vehicle, type, issue date, expiry date, attachments; each renewal is a new record

### Business rules

#### Identification
- **Identity is the internal id.** Every other module references the vehicle by id; screens show the current fleet number.
- **Fleet number** can change: unique among non-retired vehicles, kept in a searchable history, released for reuse when the vehicle is retired. No format is enforced.
- **License plate** can change (CONTRAN Res. 969/2022 forces the Mercosul format on some changes): accepts the old (`AAA9999`) and Mercosul (`AAA9A99`) formats, stored uppercase without hyphen; previous plates stay searchable; uniqueness is checked against current and previous plates of other vehicles.
- **Chassis/VIN**: exactly 17 alphanumeric characters (CONTRAN Res. 968/2022); no check-digit validation, since the Brazilian rule does not require one.
- **RENAVAM**: 11 digits; legacy 9-digit codes are left-padded with zeros. A wrong check digit is a **warning**, not an error, until the algorithm is confirmed against real RENAVAMs.
- Plate and VIN outside the expected format block saving.
- **Age** is counted from the **manufacturing year**, never the model year.

#### Accessibility
- `accessible = access_type ∈ {a, b, c} ∧ wheelchair_spaces ≥ 1`. Planning and GTFS export use only this derived flag.
- Priority seats below `max(2, ceil(10% of seats))` raise a non-compliance warning; saving is allowed.

#### Odometer
- **Monotonic odometer**: a new reading cannot be lower than the previous one. Exception: an **odometer replacement** event, which records the final km of the old unit and the initial km of the new one, keeping a continuous *accumulated km*.
- **Readings are never edited.** A wrong reading is voided with a justification and stays in the history.
- **Anomalous jump alert**: a reading implying a daily average above **N × the vehicle's average** (default 3×) **or** above an **absolute ceiling** (default 800 km/day) raises an alert; saving requires a justification.
- **Valid dates**: no reading before the service entry date, after the retirement date, or in the future.
- **Manual entry only** (single and spreadsheet) for now. The `fueling` and `telematics` sources are reserved; an ingestion interface is defined, with no adapter implemented.

#### Average km/day
- Rolling window over the **accumulated km** (so odometer replacements do not distort it): `(accumulated_km(today) − accumulated_km(today − window)) / window_days`. Days `in_maintenance` count in the denominator.
- Parameters: **window** (default 90 days), **minimum history** in days (default 14) and in km (default 500).
- Below the minimum history (e.g. a new vehicle), the average of vehicles of the same class is used and the result is flagged **estimated**.
- Used by Module 2 only for **projection** (calendar and sorting), never to decide whether an item is due (see §2.1).

#### Status and lifecycle
- **Status state machine**:
  - `in_service` ⇄ `reserve`
  - `in_service` / `reserve` → `in_maintenance` → `in_service` / `reserve`
  - any → `retired` (terminal)
- Entering/leaving `in_maintenance` is driven by opening/closing a work order that immobilizes the vehicle (Module 2); manual changes require a justification.
- **Status on a date**: the status history answers "what was the vehicle's status on date D", used by Planning for future dates and by Metrics for availability.
- **Retirement** requires a reason from the closed list, date and final km. `sale` shows an informational reminder to notify the DETRAN (CTB art. 134); `dismantling` shows a reminder to request deregistration (CTB art. 126). The reminders are not a workflow.
- A `retired` vehicle accepts no readings, work orders or schedule assignments.
- **Depot transfer** is an event with an effective date; the depot on any date comes from the transfer history, so per-depot metrics stay correct. It never changes the registration municipality or the plate.

#### Documents and eligibility
- Initial document types (the municipal and state agencies are **fictional**):

  | Document | Required for | Validity |
  |---|---|---|
  | CRLV (annual licensing) | all services | derived from the plate's last digit |
  | Tachograph verification certificate (CTB art. 105, II; Inmetro) | all services | 2 years |
  | Municipal agency inspection | `urban` | 1 year |
  | State agency registration/inspection | `intercity` | 1 year |
  | Civil liability insurance | `intercity`, `charter`, `school` | 1 year |
  | School transport authorization (CTB art. 136) | `school` | date entered |
  | Semiannual school inspection (CTB art. 136) | `school` | 6 months |

- Every type starts as `blocking`; severity is configurable per type.
- **Licensing due date** comes from a configurable table "plate last digit → deadline month" (one table, single state). Initial values from CONTRAN Res. 110/2000 (1–2 September, 3–5 October, 6–8 November, 9–0 December), flagged as not verified against the official text. The user records that the year's licensing was paid; the due date is never typed.
- Renewals never overwrite: the **current document** of a type is the latest one still valid; history is immutable.
- **Maximum age per service**: parameter in years of manufacture (seed: `urban` 10, `intercity` 15, `charter` 15, `school` 15), with an alert N months before (default 12). A charter contract may set a stricter limit (Module 4).
- **Service eligibility** is derived, never typed: a vehicle is **eligible for a service** when every `blocking` document type required by that service is current **and** the vehicle is within that service's maximum age. Planning only accepts eligible vehicles (§3.2).

#### Useful life
- **Reference useful life per class** (seed, based on the FTA useful life benchmarks: `micro`/`mini` 10 years; `midi`, `basic`, `padron`, `articulated`, `coach` 14 years), used for fleet renewal planning. Unlike maximum age, it never affects eligibility.

#### Alerts
- Each document type has an alert lead time (default 30 days). A panel lists "due soon" and "expired" documents and vehicles approaching the maximum age, by depot.
- A **daily e-mail digest per depot** (time configurable, default 07:00) goes to users with the `fleet_manager` role linked to that depot. The sent log prevents repeating the same alert; it is sent again only when an item changes from "due soon" to "expired". Uses the Module 0 notifications.

### Features
- Vehicle CRUD with filters (status, class, depot, accessibility, age, document situation, eligible service) and search by fleet number, current or previous plate, VIN and RENAVAM
- Vehicle profile: data, status timeline, fleet number/plate/depot history, km chart, documents, eligibility per service, linked work orders
- Odometer entry, single and bulk
- Documents: registration, renewal, attachments (Module 0 file storage)
- Expiry panel and daily e-mail digest
- Fleet panel: count by status, average age, % accessible, % above useful life, documents due/expired, eligible vehicles per service
- **Bulk import** (CSV and XLSX) of vehicles and odometer readings: every line is validated and errors are reported per line (line, field, value, rule); the import happens **only if there are no errors** (all or nothing); warnings such as an anomalous jump require explicit confirmation for the batch

### Out of scope for Fleet
- **Tires** → Module 2 (serialized parts)
- **Traffic fines**, **fuel consumption**, **real-time tracking** and **periodic condition assessment** → not on the roadmap
- **Vehicle allocation to routes** → Module 3

### Milestones
- **M1.1** — Vehicle registry: internal id, fleet number and plate with history, VIN/RENAVAM validation, class, capacity, structured accessibility, optional fields, service entry, depot
- **M1.2** — Odometer: monotonicity, replacement, voiding, anomalous jump, date limits, average km/day
- **M1.3** — Lifecycle: status state machine and timeline, status on a date, retirement with reason, depot transfer
- **M1.4** — Documents and eligibility: configurable types, renewals, attachments, licensing by plate digit, maximum age per service, service eligibility (requires M0.4)
- **M1.5** — Alerts: expiry panel and daily e-mail digest (requires M0.4)
- **M1.6** — Vehicle profile and fleet panel
- **M1.7** — Bulk import of vehicles and odometer readings
- **M1.8** (later) — Hour meter and onboard equipment (validator, cameras, GPS: serial number, installation/removal)

### Acceptance criteria (examples)
- A reading of 120,000 km cannot be saved if the last one was 121,500 km, except through an odometer replacement.
- A retired vehicle never appears as an option in any new schedule or work order.
- A vehicle whose plate changed from `ABC1234` to `ABC1C34` is found by searching either plate.
- A vehicle with an expired school transport authorization is not eligible for `school` but stays eligible for `urban` if its urban documents are current.
- A vehicle manufactured in 2015 is not eligible for `urban` (maximum age 10) in 2026, and the panel warned about it 12 months earlier.
- A spreadsheet with 50 vehicles and one invalid VIN imports nothing and reports the line, field, value and rule.

---

## Module 2 — Maintenance

### Goal
Plan preventive maintenance, record corrective maintenance, control parts and measure fleet reliability and cost.

### 2.1 Preventive maintenance plans

#### Entities
- **Maintenance plan**: name, applies to (chassis model / vehicle class), list of items
- **Plan item**: service (e.g. engine oil change), km interval, day interval, tolerance, expected parts and labor
- **Vehicle × plan link**: with the last execution of each item (date + km)

#### Core rule: "whichever comes first"
For each item and vehicle:

```
next_km   = last_execution_km   + interval_km
next_date = last_execution_date + interval_days

remaining_km   = next_km - current_km
remaining_days = next_date - today

km_state   = state(remaining_km,   km_alert_window)    # measured odometer only
time_state = state(remaining_days, days_alert_window)  # calendar only
item_state = worst(km_state, time_state)               # overdue > due_soon > ok
```

- **Each criterion is evaluated on absolute values** with its own alert window (e.g. 10% of the interval, or a fixed km / N days). The item state is the worse of the two, and the panel shows which criterion triggered it.
- The **average km/day** (Module 1) is used **only for projection**: `projected_km_date = today + remaining_km / avg_km_per_day`, `projected_due_date = min(next_date, projected_km_date)`. It sorts the due panel and feeds the calendar; it never changes an item's state, so a poor estimate only affects the forecast.
- An item with only one interval uses that criterion alone.
- **State**: `ok`, `due_soon` (within the alert window), `overdue`.
- Executing an item (through a preventive work order) **resets both counters** from the execution date and km, not from the theoretical due point.
- First link for a new vehicle: baseline = service entry date and initial km.

#### Features
- Plan and item management
- Due panel: list by state, sorted by `projected_due_date`, showing which criterion triggered it (km or time) and whether the projection is estimated
- Preventive work order generation from overdue/due-soon items (grouping items of the same vehicle)
- Projection: preventive maintenance calendar for the coming weeks

### 2.2 Work orders

#### Entities
- **Work order**: number, vehicle, type (`preventive`, `corrective`, `roadside`), km at opening, symptom/description, affected system (engine, brakes, suspension, electrical, body, air conditioning...), priority, immobilizes vehicle (yes/no)
- **Labor entry**: description, mechanic, hours worked, hourly rate
- **Part used**: part, quantity, unit cost (from stock at the time of issue)
- **Cause/solution** (corrective): feeds failure analysis

#### Lifecycle
```
open → in_progress ⇄ awaiting_parts → completed → closed
  └──────────────► cancelled
```
- A work order with **immobilization** sets the vehicle to `in_maintenance` when opened and releases it when closed.
- **Closing** requires exit km, labor entries and cause (corrective); once closed the work order is immutable.
- Closing a preventive work order updates the last execution of the plan items.
- **Work order cost** = Σ parts (average cost at issue) + Σ (hours × hourly rate) + third-party services.

### 2.3 Parts inventory

#### Entities
- **Part**: code, description, unit, category, fitment (compatible models), minimum stock, **reorder point**, suggested reorder quantity
- **Stock movement**: receipt (purchase, return), issue (to work order), inventory adjustment, with quantity, cost and reference

#### Rules
- Balance = Σ movements (never edited directly).
- **Weighted average cost** recalculated on every receipt.
- An issue cannot result in a negative balance.
- **Reorder alert** when `balance ≤ reorder_point`; optionally accounting for parts reserved for open work orders and expected consumption from upcoming preventive maintenance.
- Stock count: generates an adjustment movement with justification.

### 2.4 Maintenance metrics

| Metric | Formula | Breakdown |
|--------|---------|-----------|
| **MTBF** | hours (or km) in service ÷ number of corrective failures | vehicle, model, system, period |
| **MTTR** | Σ immobilization time in corrective work orders ÷ number of corrective work orders | same |
| **Availability** | (period hours − immobilized hours) ÷ period hours | fleet, depot, model |
| **Cost per km** | total work order cost ÷ km driven in the period | vehicle, model |
| **% preventive × corrective** | cost or number of work orders by type | period |
| **Backlog** | open work orders by age and priority | snapshot |

- Define what counts as a "failure": only `corrective` and `roadside` work orders, excluding cancelled ones.
- Km driven in the period comes from Module 1 (odometer difference).

### Milestones
- **M2.1** — Corrective work orders (lifecycle + effect on vehicle status)
- **M2.2** — Parts inventory: registry, movements, average cost, issue to work orders
- **M2.3** — Reorder point alerts
- **M2.4** — Preventive plans with the "km or time, whichever comes first" rule
- **M2.5** — Due panel and preventive work order generation
- **M2.6** — Metrics (MTBF, MTTR, availability, cost/km)

### Acceptance criteria (examples)
- An item with a 10,000 km / 180 day interval, executed 170 days and 4,000 km ago, shows as `due_soon` by the **time** criterion.
- Opening a work order with immobilization immediately removes the vehicle from the list available for scheduling.
- Issuing 3 units with a balance of 2 is rejected.

### Open questions
- Labor: single hourly rate, per mechanic, or per specialty?
- Are third-party services (external workshop) part of the MVP?
- Does overdue preventive maintenance block scheduling or only warn? (affects Module 3)

---

## Module 3 — Operations Planning

### Goal
Define the service offering (routes, itineraries, timetables) and assign vehicles and drivers to it, ensuring no assignment violates operational or labor rules.

### 3.1 Routes, itineraries and timetables

#### Entities
- **Route**: code, name, **service** (`urban`, `intercity`, `charter`, `school`; exactly one per route), municipality(ies) served, vehicle requirements (minimum class, mandatory accessibility)
- **Itinerary**: route, direction (outbound/inbound/loop), sequence of stops/streets, length in km, running time per time band
- **Stop**: name, coordinates (optional), terminal yes/no
- **Timetable**: route, day type (weekday, Saturday, Sunday/holiday), validity period, list of **trips** (departure time, direction, itinerary)

#### Rules
- Day type resolved from the calendar + holiday registry (Module 0).
- Only one active timetable per route and day type on any given date.
- Arrival time = departure + running time for the time band.

### 3.2 Blocks and scheduling

#### Concepts
- **Block (vehicle duty)**: chain of trips the same vehicle runs during the day, including depot runs (deadhead km).
- **Shift (driver duty)**: sequence of trips/blocks assigned to a driver, with start, end and breaks.
- **Daily schedule**: for a given date, assignment of a vehicle to each block and a driver to each shift.

#### Additional entities
- **Driver**: employee ID, name, driver's license (category, expiry), mandatory training (public transport, school transport) with expiry, depot, status (active, on vacation, on leave)
- **Driver unavailability**: vacation, day off, sick leave, with period

#### Conflict validations
Each assignment is checked against:

**Vehicle**
- Status must be `in_service` or `reserve` on the date — a vehicle `in_maintenance` or `retired` is **blocked**.
- Cannot be in two blocks with overlapping times.
- Must meet the route requirements (minimum class, capacity, accessibility).
- Must be **eligible for the route's service** on the date (Module 1: current blocking documents and within the maximum age). When the route belongs to a charter contract with its own maximum age, the stricter of the service and contract limits applies.
- Overdue preventive maintenance: block or warn, depending on a parameter.

**Driver**
- Cannot have overlapping shifts.
- **Minimum rest between shifts** respected — parameter, e.g. 11 h.
- Maximum daily working time and intra-shift break — parameters.
- Driver's license with a compatible category (D for buses) and valid on the date.
- Mandatory training valid (e.g. school transport for school charter routes).
- No registered unavailability on the date.

Validations return **errors (blocking)** and **warnings (allowed with justification)**, both explaining the rule that was violated.

### 3.3 Execution (actuals)
Bridge between the plan and Charter/Metrics.
- **Completed trip** record: actual departure/arrival time, actual vehicle and driver, km, incidents.
- Trip status: `completed`, `completed_late`, `not_operated` (with reason: breakdown, no driver, traffic...), `replaced`.
- A vehicle swap during the day (e.g. breakdown) creates an incident and may open a roadside work order (Module 2).

### Features
- Route, itinerary and timetable management
- Block generation from the timetable (manual at first; automatic chaining later)
- Daily schedule screen with real-time validation and suggestions of available vehicles/drivers
- Copy a schedule from one day to another of the same day type
- Actuals entry and daily fulfillment panel

### Milestones
- **M3.1** — Routes, stops and itineraries
- **M3.2** — Timetables by day type and validity period
- **M3.3** — Driver registry and unavailability
- **M3.4** — Blocks and daily vehicle scheduling with availability validation
- **M3.5** — Driver scheduling with working time and rest validation
- **M3.6** — Actuals recording and incidents
- **M3.7** (later) — Automatic block/shift generation (optimization)

### Acceptance criteria (examples)
- A driver whose shift ends at 23:00 cannot be scheduled at 07:00 the next day with an 11 h minimum rest configured; the message reports the missing rest (3 h).
- A vehicle without accessibility cannot be assigned to a route that requires it.

### Open questions
- How detailed must the itinerary be: is a list of stops enough, or do we need a map/geometry?
- Labor rules: which parameters are in the MVP (rest between shifts, maximum working time, intra-shift break, overtime)?
- Actuals entered manually, imported (spreadsheet), or both?

---

## Module 4 — Charter

### Goal
Manage charter contracts with municipalities, measure monthly what was delivered, apply SLA penalties and generate the billing report.

### 4.1 Contracts

#### Entities
- **Contract**: number, contracting municipality, scope (e.g. school transport), validity (start/end), estimated total value, price adjustment index and base date, status (`active`, `suspended`, `terminated`)
- **Amendment**: term, value, addition/removal of routes, with effective date
- **Contracted route**: contract, code, description, itinerary (Module 3), km per trip, planned trips per day/day type, minimum vehicle class, maximum vehicle age (optional, stricter than the service limit), requirements (accessibility, attendant, driver school transport training)
- **Pricing model** (per route):
  - **per km**: rate/km × km completed
  - **per trip**: rate/trip × trips completed
  - (later) fixed monthly fee, or mixed
- **SLA rules**: delay tolerance, penalty per trip not operated, per delay, per non-compliant vehicle, deduction cap

#### Rules
- Rates and rules in effect are resolved **by trip date** (an amendment or price adjustment in the middle of the month splits the calculation).
- A contract outside its validity period accepts no trips or measurements.
- Charter routes are Module 3 routes linked to the contract — they reuse scheduling and actuals.

### 4.2 Monthly measurement

#### Flow
```
draft → under_review → approved → invoiced
```
1. **Calculation**: for the contract and month, consolidates planned vs. completed trips from the actuals (Module 3).
2. **Review**: the operator reviews, justifies discrepancies and adjusts (with audit trail).
3. **Approval**: freezes the measurement (immutable). Later corrections only through a supplementary measurement.
4. **Invoicing**: generates the report.

#### Calculation
```
gross_amount = Σ per route (completed_units × unit_rate_in_effect)
deductions   = Σ SLA penalties (capped)
net_amount   = gross_amount − deductions
```

### 4.3 SLA control
- **Trip not operated**: planned with no matching actual, or marked `not_operated`.
- **Delay**: actual departure − planned departure > contract tolerance.
- **Non-compliant vehicle**: vehicle that does not meet the route requirements.
- Metrics: % fulfillment, % punctuality, accumulated deductions by contract/route/month.
- Reason classification (company's responsibility × external) — external reasons may be exempt from deductions depending on the contract.

### 4.4 Billing report
- Per contract and month: summary (gross, deductions, net), breakdown by route, list of trips not operated/late with reason
- Export to PDF and spreadsheet
- Report number and version tied to the approved measurement

### Milestones
- **M4.1** — Contracts, amendments and contracted routes
- **M4.2** — Per-km and per-trip pricing with validity periods
- **M4.3** — Measurement calculation from actuals
- **M4.4** — SLA rules and deduction calculation
- **M4.5** — Review and approval flow (immutable measurement)
- **M4.6** — Billing report (PDF/spreadsheet)
- **M4.7** (later) — Automatic index-based price adjustment, supplementary measurement

### Acceptance criteria (examples)
- A route with 44 planned trips in the month and 42 completed, priced per trip at R$ 300.00 with a penalty of 2× the trip rate per trip not operated: gross R$ 12,600.00, deductions R$ 1,200.00, net R$ 11,400.00.
- An approved measurement cannot be changed; the edit button does not exist and the API rejects it.

### Open questions
- Is the penalty about the trip not operated (which is already not paid) or an additional fine? Define the default model for the seed.
- Holidays and the school calendar change planned trips — do we need a calendar per contract?
- Does the report follow a standard municipal template?

---

## Module 5 — Metrics and dashboards

### Goal
Consolidate the operational view for managers in a single place.

### Scope
- **Fleet dashboard**: vehicles by status, availability, average age
- **Maintenance dashboard**: MTBF, MTTR, cost/km, overdue preventive maintenance, work order backlog, parts below reorder point
- **Operations dashboard**: trip fulfillment, punctuality, incidents of the day
- **Charter dashboard**: billing per contract, deductions, SLA per municipality
- Common filters: period, depot, model, route, contract
- Data export for each chart

### Milestones
- **M5.1** — Fleet and maintenance dashboard
- **M5.2** — Operations dashboard
- **M5.3** — Charter dashboard
- **M5.4** (later) — Period comparisons and targets

---

## Suggested milestone sequence

Order that delivers demonstrable value early and respects dependencies:

1. **M0.1 → M0.4** — Foundation (M0.4 notifications and file storage must precede M1.4)
2. **M1.1 → M1.7** — Fleet (M1.1 → M1.4 must precede M3.4: scheduling needs service eligibility)
3. **M2.1 → M2.3** — Corrective work orders and inventory
4. **M2.4 → M2.5** — Preventive maintenance ("whichever comes first" rule)
5. **M3.1 → M3.5** — Planning and scheduling with validations
6. **M3.6** — Actuals
7. **M4.1 → M4.6** — Full charter module
8. **M2.6 + M5.x** — Metrics and dashboards
9. Later items (M0.5, M1.8, M3.7, M4.7, M5.4)

---

## Pending technical decisions

- Frontend and database stack (backend: Java 21 + Spring Boot 4, in `server/`)
- Testing strategy for domain rules
- Report generation (PDF/spreadsheet)
- Hosting and CI/CD
