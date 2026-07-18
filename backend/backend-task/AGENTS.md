# AGENTS.md

This directory contains the implementation plan for the backend build-out of the Employee Management System.

## Local System Prompt for Agents

1. Read `backend/backend.AGENTS.md` first.
2. Read `backend/backend-task/taskTracker.md` before starting any numbered task.
3. Read the numbered task file being implemented in full before changing code.
4. Implement one numbered task per pass unless the user explicitly requests a batch.
5. Keep mandatory tasks in dependency order.
6. Do not mark a task `Completed` until implementation, verification, and evidence are recorded.
7. Update both the numbered task file and `backend/backend-task/taskTracker.md` in the same completion turn.
8. Record exact backend commands used for type checking, building, Prisma validation, migrations, seeding, and tests.
9. If verification fails, keep the task `In Progress` or mark it `Blocked`; never present a failing task as complete.
10. Do not silently weaken role rules, API contracts, cookie behavior, or validation requirements to make a task easier.
11. Update backend docs and `.env.example` files whenever an API contract, env contract, or reviewer workflow changes.

## Scope Rules

- The backend is the only security boundary for auth, RBAC, field-level authorization, and hierarchy validation.
- All API routes must live under `/api`.
- Authentication uses JWT in an HTTP-only cookie, not local storage and not Supabase Auth.
- One `Employee` model owns profile data, login credentials, role, status, and reporting hierarchy.
- Deleted employees are soft-deleted with `deletedAt` and excluded from normal reads.
- `SUPER_ADMIN`, `HR_MANAGER`, and `EMPLOYEE` are fixed enums and must not be replaced with arbitrary strings.
- Frontend permission checks are UX only; every sensitive backend route must enforce authorization independently.

## Verification Rules

- Prefer `npm run typecheck`, `npm run build`, and focused backend tests once those scripts exist.
- Use `npx prisma validate` after schema work and migration-related commands after Prisma changes.
- Record `Not Run` when a command does not exist yet instead of inventing a passing result.
- When a task is backend-only, set any unrelated verification slot to `N/A`.
- Keep completion evidence factual and command-specific.

## Completion Evidence Template

Every numbered task ends with:

```text
Status:
Implemented files:
Typecheck command:
Typecheck result:
Build command:
Build result:
Backend test command:
Backend test result:
Manual verification:
Evidence:
Caveats:
```
