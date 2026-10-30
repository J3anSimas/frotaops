# ADR 0001 — Database and migration tool

- **Status**: Accepted
- **Date**: 2026-10-30
- **Issue**: #1 (`M0.1-01`)

## Context

The ROADMAP left the database open (*Pending technical decisions*), and nothing persistent can start without it. The choice must support:

1. **Immutable histories**: odometer readings, status events, plate and fleet number changes, documents, stock movements and approved measurements are never edited, only corrected through new entries (ROADMAP *Principles*).
2. **"State on a date" queries**: status, depot and parameter value on date D (M1.3, M0.2-05), used by Planning for future dates and by Metrics for past periods.
3. **Validity without overlap**, which appears across modules:
   - one active timetable per route and day type on any date (M3.2);
   - a vehicle in at most one block at a time (M3.4);
   - driver shifts that do not overlap (M3.5);
   - contract and amendment validity periods (M4.1).
4. **Search by current or previous plate / fleet number**, and plate uniqueness against the current and previous plates of *other* vehicles (M1.1).
5. **Local execution with Docker**, and tests against the same engine as production.

The backend is Java 21 + Spring Boot 4 (`server/`). Hosting for the demo is still undecided, so the database must be widely available as a managed service.

## Options considered

### Database

| | PostgreSQL | MySQL / MariaDB | Oracle Database Free |
|---|---|---|---|
| Append-only tables (permissions + trigger) | Yes | Yes | Yes |
| Range types (`daterange`, `tstzrange`) | Yes | No | No |
| Non-overlap enforced by the database | Yes: `EXCLUDE USING gist`; `WITHOUT OVERLAPS` on PK/UNIQUE since 18 | No: application only, open to race conditions | Partial: temporal validity, without non-overlap constraints |
| Partial and fuzzy text search | `pg_trgm` | `FULLTEXT`, limited for partial plates | Oracle Text |
| Local Docker | Small official image | Small official image | Large image; the Free edition caps CPU, RAM and data size |
| Managed hosting for the demo | Widely available, including free tiers | Widely available | Limited |

### Migration tool

| | Flyway | Liquibase |
|---|---|---|
| Format | Plain versioned SQL (`V1__description.sql`) | XML / YAML / SQL changesets |
| PostgreSQL-specific DDL (`EXCLUDE`, triggers, extensions) | Written directly | Needs raw SQL changesets anyway |
| Multi-database abstraction | No | Yes, not needed here |
| Rollback | Forward-only (paid undo) | Generated rollback |
| Spring Boot integration | Auto-configured | Auto-configured |

## Decision

- **PostgreSQL, version 18 or later.** It is the only option that enforces validity without overlap in the database, which several modules need. Version 18 adds `WITHOUT OVERLAPS` to primary and unique keys, which keeps temporal constraints short.
- **Flyway** with plain SQL migrations. There is one database, so a multi-database abstraction adds nothing. The PostgreSQL-specific DDL reads best as SQL. Forward-only migrations match the project's correct-by-new-entry principle.
- **Immutability is enforced in two places.** The domain is the primary guard and is tested in isolation. The database is the safety net: append-only tables reject `UPDATE` and `DELETE` through permissions and a trigger.
- **Tests use PostgreSQL, never H2.** H2 has no range types or exclusion constraints, so tests on it would pass against a different engine. How the persistence tests run (e.g. Testcontainers) is part of ADR 0002.

## Consequences

### Schema conventions
- **History as explicit tables.** Readings, status events, depot transfers and plate / fleet number changes are business entities in the ROADMAP, so each one gets its own append-only table. There are no generic system-versioned tables. "State on date D" is the latest event with `effective_at ≤ D`, served by an index on `(vehicle_id, effective_at DESC)`.
- **Corrections are new rows.** Rows in append-only tables are never updated, not even to mark them void. For example, voiding an odometer reading (M1.2) inserts a void entry that references the reading and carries the justification. The reading row itself does not change.
- **Freeze on a state change.** Some records are editable until a transition and frozen after it, such as a measurement after approval (M4.5) or a work order after closing (M2.2). For these, the trigger rejects changes once the row reaches the frozen state, not on every update.
- **Non-overlap as constraints.** Validity periods are stored as ranges and protected by an exclusion constraint or `WITHOUT OVERLAPS`, both of which need the `btree_gist` extension. Example: a plate may never belong to two different vehicles, while the same vehicle may get an old plate back:

  ```sql
  EXCLUDE USING gist (plate WITH =, vehicle_id WITH <>)
  ```

  The domain still validates first and returns the error message the ROADMAP requires (offending value and expected shape). The constraint only catches what slips past, such as two concurrent requests.
- **Types**: `timestamptz` for instants and `date` for calendar dates, such as document expiry and holidays.
- Extensions enabled by the first migration: `btree_gist`, plus `pg_trgm` once partial plate / fleet number search arrives (M1.1).

### Process
- Applied migrations are never edited. Flyway validates their checksums, so every change is a new migration.
- Managed hosting must offer PostgreSQL 18 or later. If the chosen provider does not, `EXCLUDE USING gist` gives the same guarantees on older versions with more verbose DDL.

### Backend changes (for the developer)
These changes are described here, not applied, because `server/` is human-owned:
- Dependencies: the PostgreSQL JDBC driver (`org.postgresql:postgresql`, runtime), `spring-boot-starter-flyway` and `org.flywaydb:flyway-database-postgresql`. Since Flyway 10, each database's support ships as a separate module.
- Migrations go in `server/src/main/resources/db/migration/`, Flyway's default location.
- A local PostgreSQL in Docker (compose file), with the connection in `application.yaml` per profile.

### Left to other decisions
- Data access style (JPA, Spring Data JDBC or jOOQ) and the persistence test setup: **ADR 0002** (#2).
- Hosting and CI/CD: ROADMAP *Pending technical decisions*.
