# Valoura Current State

## 1. Purpose

This document records the actual starting state of the fresh Valoura implementation.

Previous Valoura development work may be used as reference and learning material, but no previously implemented feature is considered complete unless it is implemented and verified in this repository.

---

## 2. Current Project Status

Valoura is starting from a fresh implementation baseline.

The repository and project planning structure have been established, but application development has not started yet.

Current stage:

- Phase 0 — Establish actual starting point
- Milestone: M0 — Foundation
- Active backlog issue: V-01
- Application implementation: Not started

---

## 3. Architecture Baseline

The fresh Valoura implementation will use the following baseline architecture:

| Area | Technology |
|---|---|
| Frontend | Next.js + React + TypeScript |
| Styling | Tailwind CSS |
| Backend | Node.js + Express + TypeScript |
| Database | PostgreSQL |
| Validation | Zod |
| Repository | npm workspaces monorepo |

Additional architectural decisions will be documented before their implementation when required.

---

## 4. Database Decision

PostgreSQL is the selected database for the fresh Valoura implementation.

Previous versions or planning documents may contain references to MongoDB or Mongoose. Those references do not represent the database architecture for the new implementation.

Any conflicting documentation must be updated before the related feature is implemented.

No PostgreSQL database connection has been implemented yet.

---

## 5. Existing Implementation

No application functionality from the previous Valoura implementation is being treated as completed.

Previous work may be consulted for:

- learning;
- implementation reference;
- UI inspiration;
- identifying previous mistakes;
- improving the new implementation.

Previous code must not be copied into the new implementation without review.

---

## 6. Current State by Area

| Area | Status | Evidence / Notes |
|---|---|---|
| Git repository | Working | Repository initialized and connected to GitHub |
| GitHub planning | Working | Milestones, labels and project board configured |
| Project documentation | Working | Core documents stored under `docs/` |
| Monorepo workspace | Missing | Planned for V-02 |
| Next.js web application | Missing | Not scaffolded |
| Express API | Missing | Not scaffolded |
| Worker application | Missing | Not scaffolded |
| PostgreSQL integration | Missing | Architecture decision recorded only |
| Zod validation | Missing | Not implemented |
| Authentication | Missing | Not implemented |
| Customer functionality | Missing | Not implemented |
| Vendor functionality | Missing | Not implemented |
| Admin functionality | Missing | Not implemented |
| Vendor discovery | Missing | Not implemented |
| Booking | Missing | Not implemented |
| Payments | Missing | Not implemented |
| Automated testing | Missing | Not configured |
| CI/CD | Missing | Not configured |
| Production deployment | Missing | Not configured |

---

## 7. Reusable Work

The following work is reusable for the fresh implementation:

- Valoura product requirements and planning documentation;
- implementation workflow;
- Git/GitHub workflow conventions;
- issue and pull request templates;
- GitHub milestones;
- project board structure;
- project labels;
- lessons learned from the previous implementation.

Previous application code is reference material only until reviewed and intentionally reintroduced.

---

## 8. Known Gaps

The project currently has no application scaffold.

The immediate gaps are:

1. npm workspaces have not been configured.
2. The Next.js web application has not been scaffolded.
3. The Express API has not been scaffolded.
4. The worker application has not been scaffolded.
5. Environment validation has not been configured.
6. PostgreSQL integration has not been implemented.

These gaps are expected at this stage.

---

## 9. Next Incomplete Prerequisite

The next prerequisite after V-01 is:

**V-02 — Scaffold npm workspaces, web/API/worker, and environment validation.**

V-02 should begin only after V-01 has been reviewed and merged.

---

## 10. Phase 0 Conclusion

The project is intentionally starting from a clean application baseline.

No authentication, database integration, frontend feature, vendor feature, booking feature, or payment feature is considered implemented.

Phase 0 establishes the source of truth from which subsequent Valoura implementation work will proceed.