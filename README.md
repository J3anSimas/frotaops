# FrotaOps

Fleet management and transport operations platform for a **fictional** bus company.

> Demo project. All data (vehicles, drivers, routes, municipalities and contracts) is artificially generated; no real data is used.

## What the system covers

| Module | Description |
|--------|-------------|
| **Fleet** | Vehicle registry (fleet number, plate, chassis, class, capacity, accessibility) across multiple depots, odometer history, status (in service, in maintenance, reserve, retired), documents with expiry alerts and service eligibility (urban, intercity, charter, school) |
| **Maintenance** | Preventive plans by km or time (whichever comes first), corrective work orders, parts inventory with reorder points, and metrics such as MTBF, cost per km and fleet availability |
| **Operations Planning** | Routes, itineraries, timetables, and vehicle and driver scheduling with conflict validation |
| **Charter** | Contracts with municipalities, monthly measurement of completed trips, SLA control and billing report |
| **Metrics** | Consolidated dashboards for fleet, maintenance, operations and charter |

## Key business rules

- **Preventive maintenance, "whichever comes first"**: each plan item has a km interval and a day interval, each checked against measured values; the item is due when either criterion is reached. The vehicle's average km/day only projects the due date.
- **Derived service eligibility**: a vehicle may run a service (urban, intercity, charter, school) only while every blocking document that service requires is current and the vehicle is within the service's maximum age.
- **Scheduling with conflict validation**: a vehicle in maintenance, retired or not eligible for the route's service cannot be scheduled; drivers must respect minimum rest between shifts, maximum working time, and valid license and training.
- **Immutable charter measurement**: once approved, a measurement is frozen; corrections only through a supplementary measurement.
- **Auditable history**: odometer, inventory and measurements change through entries, never through direct edits.

## Roadmap

The detailed per-module plan, with entities, rules, milestones and open questions, is in [ROADMAP.md](ROADMAP.md). How the work is planned, tracked and accepted is in [docs/process.md](docs/process.md).

## Repository structure

```
frotaops/
├── server/       # Backend — Java 21, Spring Boot 4, Maven
├── README.md
└── ROADMAP.md
```

## Getting started

### Server

Requires Java 21.

```bash
cd server
./mvnw spring-boot:run
```
