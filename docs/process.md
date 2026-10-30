# Planning and delivery process

How FrotaOps work is planned, tracked and accepted. The *what* lives in [ROADMAP.md](../ROADMAP.md); this document covers the *how*.

## Method

A lightweight hybrid of the **Linear Method** and **Kanban**, sized for a single developer and tracked in **GitHub Projects**, so the plan stays public and linked to the code:

- **2-week cycles** give a planning and review rhythm.
- **WIP limit of 2**: at most two issues `In Progress` at once, so work flows to done instead of piling up half-finished.
- **Vertical slices**: every issue delivers one working rule end to end (domain, persistence, API, tests), never a single layer.
- **Explicit quality gates**: a Definition of Ready before work starts and a Definition of Done before an issue closes.

Scrum ceremonies built for teams (daily stand-up, separate PO/SM roles) are intentionally left out.

## Roles

| Who | Responsibilities |
|---|---|
| Developer (human) | Prioritizes, plans cycles, writes all code under `server/`, closes issues |
| Agent | Drafts issue descriptions from the ROADMAP, maintains docs, reviews the code of each issue against its acceptance criteria and the Definition of Done |

See [AGENTS.md](../AGENTS.md) for the agent's boundaries.

## Mapping the roadmap to GitHub

| ROADMAP | GitHub |
|---|---|
| FrotaOps | One **Project** (board) for the whole roadmap |
| Module (0–5) | Project field **Module** (single select) |
| Milestone (M1.1, M1.2…) | Repository **milestone** |
| Deliverable slice within a milestone | **Issue** (native sub-issues when needed) |
| "Blocked by" | Native issue **dependencies** |
| Acceptance criteria | Task list in the issue body |
| Rhythm | Project field **Iteration** (2 weeks) |
| Estimate | Project field **Size** |
| End-of-cycle review | Project **status update** |

A single Project with a Module field, rather than one Project per module, keeps cross-module blockers visible (e.g. M1.4 blocking M3.4).

Pull requests reference the issue they deliver (`Closes #12`), so every acceptance criterion traces to the code that satisfies it.

### Labels
- **Type** (exactly one): `feature`, `bug`, `tech-debt`, `chore` (build, CI, tooling), `docs`, `spike`

### Workflow states
Project field **Status**: `Backlog` → `Todo` (in the current iteration) → `In Progress` (WIP limit 2) → `In Review` (agent review) → `Done`. Abandoned issues are closed as *not planned*.

An issue leaves `In Review` only when the review has no blocking findings.

## Estimates

Project field **Size**:

| Size | Effort |
|---|---|
| XS | ~2 h |
| S | ~half a day |
| M | 1–2 days |
| L | 3–5 days, **must be split into sub-issues before entering a cycle** |

There is no XL. Only issues of size XS to M enter a cycle; an L parent stays out and its sub-issues are scheduled instead.

## Issue anatomy

Title, milestone, label, Module, Size and blockers live in GitHub's own fields; the body holds the rest:

```markdown
**Context**
Why this exists, with a pointer to the ROADMAP section.

**Scope**
What is delivered.

**Out of scope**
What is deliberately left to other issues.

**Acceptance criteria**
- [ ] Given <state>, when <action>, then <observable result>.

**References**
ROADMAP section, regulation, research notes.
```

- Acceptance criteria use **Given / When / Then** so they map almost one to one to test names.
- Error criteria state the offending value and the expected shape.
- Titles describe behavior ("Change plate with history and search by previous plate"), not layers ("Create PlateHistory repository").

### Spikes
A spike answers a question instead of delivering behavior. It has a **timebox** and its output is a document (usually an ADR under `docs/adr/`), never production code. If the timebox ends, record what is missing and decide with the information at hand.

### Bugs
A bug issue states the observed behavior, the expected behavior and the steps to reproduce. Its fix ships with a regression test.

## Definition of Ready

An issue can enter a cycle when:

- it is XS, S or M;
- it has context, scope, out of scope and acceptance criteria;
- its blockers are done or scheduled earlier in the same cycle;
- it points to the ROADMAP section it comes from.

## Definition of Done

An issue is done when:

- every acceptance criterion is covered by a passing test, and the whole suite runs with a single command;
- the agent review has no blocking findings;
- error messages include the offending value and the expected shape;
- the ROADMAP and docs reflect any rule that changed;
- the seed shows the behavior, when it matters for the demo.

## Cycle routine

**Planning (start of cycle)**
1. Close or carry over the previous cycle's leftovers.
2. Pull issues from the backlog in milestone order, respecting blockers, until the cycle is full.
3. Check every pulled issue against the Definition of Ready.

**Review (end of cycle)**: posted as a **status update** on the GitHub Project:

```markdown
**Cycle <n> — <dates>**
Delivered: <issues and milestones completed>
Slipped: <issues carried over, and why>
Learned: <what changed in our understanding of the domain or the plan>
Next: <focus of the next cycle>
```

## Backlog depth

Only the modules being worked on next are broken into issues. Later modules exist as repository milestones with no issues yet, and are detailed when they come up, so the plan does not freeze decisions that will change.

Draft backlogs are written under `docs/backlog/` for review before they are created on GitHub.
