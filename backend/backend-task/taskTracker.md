# Backend Task Tracker

This is the status source for the backend implementation plan. The numbered files contain the executable task scope and completion evidence.

## Status Rules

Allowed task statuses:

- `Not Started`
- `In Progress`
- `Blocked`
- `Completed`

Allowed verification statuses:

- `Not Run`
- `Passed`
- `Failed`
- `Blocked`
- `N/A`

## Confirmed Current Backend Baseline

- `backend/` currently contains `package.json` and `backend.AGENTS.md`; the Express app, Prisma schema, and source structure do not exist yet.
- `backend/package.json` is still the default placeholder and does not provide backend build, dev, Prisma, or typecheck scripts.
- No `src/`, `prisma/`, `.env.example`, or test files exist yet in `backend/`.
- The repository instructions already define the required backend stack, API surface, RBAC rules, hierarchy rules, and response format.
- The task list below is intentionally chronological so the backend can be implemented one module at a time without inventing architecture during later tasks.

## Mandatory Tasks

| # | Task | Status | Depends On | Typecheck | Build | Backend Tests | Completion Evidence |
|---:|---|---|---|---|---|---|---|
| 0 | [Backend Bootstrap and Tooling Baseline](./0.%20Backend%20Bootstrap%20and%20Tooling%20Baseline.md) | Not Started | None | Not Run | Not Run | Not Run | Add evidence in Task 0. |
| 1 | [Environment Validation, Express Skeleton, and Prisma Client Wiring](./1.%20Environment%20Validation,%20Express%20Skeleton,%20and%20Prisma%20Client%20Wiring.md) | Not Started | Task 0 | Not Run | Not Run | Not Run | Add evidence in Task 1. |
| 2 | [Prisma Employee Schema and Initial Migration](./2.%20Prisma%20Employee%20Schema%20and%20Initial%20Migration.md) | Not Started | Task 1 | Not Run | Not Run | Not Run | Add evidence in Task 2. |
| 3 | [Seed Data and Reviewer Accounts](./3.%20Seed%20Data%20and%20Reviewer%20Accounts.md) | Not Started | Task 2 | Not Run | Not Run | Not Run | Add evidence in Task 3. |
| 4 | [Shared API Responses, Error Handling, and Security Middleware](./4.%20Shared%20API%20Responses,%20Error%20Handling,%20and%20Security%20Middleware.md) | Not Started | Task 1 | Not Run | Not Run | Not Run | Add evidence in Task 4. |
| 5 | [Authentication Endpoints and Session Cookie Flow](./5.%20Authentication%20Endpoints%20and%20Session%20Cookie%20Flow.md) | Not Started | Tasks 2, 3, 4 | Not Run | Not Run | Not Run | Add evidence in Task 5. |
| 6 | [Authentication Middleware and RBAC Guards](./6.%20Authentication%20Middleware%20and%20RBAC%20Guards.md) | Not Started | Tasks 4, 5 | Not Run | Not Run | Not Run | Add evidence in Task 6. |
| 7 | [Employee Validation, Serialization, and Query Contracts](./7.%20Employee%20Validation,%20Serialization,%20and%20Query%20Contracts.md) | Not Started | Tasks 2, 4, 6 | Not Run | Not Run | Not Run | Add evidence in Task 7. |
| 8 | [Employee List and Create Endpoints](./8.%20Employee%20List%20and%20Create%20Endpoints.md) | Not Started | Task 7 | Not Run | Not Run | Not Run | Add evidence in Task 8. |
| 9 | [Employee Detail, Update, and Self-Edit Restrictions](./9.%20Employee%20Detail,%20Update,%20and%20Self-Edit%20Restrictions.md) | Not Started | Task 8 | Not Run | Not Run | Not Run | Add evidence in Task 9. |
| 10 | [Employee Soft Delete and Manager Reassignment](./10.%20Employee%20Soft%20Delete%20and%20Manager%20Reassignment.md) | Not Started | Task 9 | Not Run | Not Run | Not Run | Add evidence in Task 10. |
| 11 | [Manager Assignment, Circular Reporting Protection, and Organization Reads](./11.%20Manager%20Assignment,%20Circular%20Reporting%20Protection,%20and%20Organization%20Reads.md) | Not Started | Tasks 7, 9 | Not Run | Not Run | Not Run | Add evidence in Task 11. |
| 12 | [Dashboard Summary and API Health](./12.%20Dashboard%20Summary%20and%20API%20Health.md) | Not Started | Tasks 6, 8, 10, 11 | Not Run | Not Run | Not Run | Add evidence in Task 12. |
| 13 | [Regression Verification, Env Examples, and Backend Documentation](./13.%20Regression%20Verification,%20Env%20Examples,%20and%20Backend%20Documentation.md) | Not Started | Tasks 0-12 | Not Run | Not Run | Not Run | Add evidence in Task 13. |

## Implementation Order

Recommended chronological order:

1. Tasks 0-4
2. Tasks 5-7
3. Tasks 8-10
4. Tasks 11-12
5. Task 13

Task 4 can start after Task 1 if the folder structure exists. Do not start employee endpoints before Task 7 establishes validation and serialization contracts. Do not start final documentation before the backend contract is actually implemented.

## Expected Final Coverage

By the end of Task 13, the backend should cover:

- Prisma schema, migration, and seed data
- secure login, logout, and `auth/me`
- JWT cookie auth and backend-enforced RBAC
- employee CRUD with field-level restrictions
- soft delete behavior
- manager assignment with circular-reporting prevention
- direct reportees and organization tree responses
- dashboard summary metrics
- centralized validation, error handling, and response helpers
- `.env.example`, README-level backend docs, and reviewer-ready verification evidence
