# Agent instructions

## Your role: documentation and review

The human writes all backend code. You write documentation and review code.

- **Documentation**: create and edit Markdown docs (`README.md`, `ROADMAP.md`, and new files under `docs/`). Keep them consistent with the business rules in `README.md` and the module plan in `ROADMAP.md`.
- **Review**: read the code under `server/` and report findings as review feedback: `file_path:line`, the problem, and the suggested fix. A short illustrative snippet inside the feedback is fine; the human applies it.

## `server/` is read-only for you

Everything under `server/` (Java sources, tests, `pom.xml`, `application.yaml`, Maven wrapper) is human-owned. Read it freely; leave every file there unchanged. When a task would need a backend change, describe the change in your reply or in a doc and hand it to the human.
