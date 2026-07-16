# AGENTS.md — Employee Management System

## 1. Purpose

This file contains repository-wide instructions for building the Employee Management System hiring assignment.

The goal is to deliver a polished, complete, and easy-to-review assignment within the expected **8–10 hour scope**. Favor correctness, clarity, and visible completion over unnecessary production infrastructure.

This is a monorepo containing two independently deployable applications:

```text
employee-management-system/
├── AGENTS.md
├── README.md
├── frontend/
│   ├── AGENTS.md
│   └── ...
└── backend/
    ├── AGENTS.md
    └── ...
```

The root `AGENTS.md` applies to the entire repository. The `frontend/AGENTS.md` and `backend/AGENTS.md` files add folder-specific instructions. When rules conflict, the instructions closest to the file being edited take priority.

---

## 2. Repository and Deployment Decision

Use **one GitHub repository** for the complete assignment.

Deploy the two applications separately from the same repository:

- `frontend/` → Vercel
- `backend/` → Render
- PostgreSQL database → Supabase

Configure the service root directory during deployment:

```text
Vercel root directory: frontend
Render root directory: backend
```

Do not split the project into two GitHub repositories.

Do not introduce Turborepo, Nx, shared workspace packages, or another monorepo framework. They are unnecessary for this assignment.

The frontend and backend may each have their own:

- `package.json`
- lockfile
- environment variables
- build command
- deployment configuration

A root `package.json` with convenience scripts is optional, not required.

---

## 3. Assignment Objective

Build an Employee Management System with:

- secure login and logout
- protected routes
- JWT authentication
- bcrypt password hashing
- role-based access control
- employee CRUD
- dashboard statistics
- organizational reporting hierarchy
- circular-reporting prevention
- search, filtering, sorting, and pagination
- frontend and backend validation
- responsive UI
- clean documentation

The final result should feel complete and hiring-ready, but it should remain appropriately scoped for a take-home assignment.

---

## 4. Required Technology Stack

### Frontend

- Next.js using the App Router
- React
- TypeScript with strict mode
- Tailwind CSS
- shadcn/ui
- TanStack Query
- React Hook Form
- Zod
- Axios
- Sonner
- Lucide React
- date-fns

Optional only when needed:

- `next-themes` for dark mode
- shadcn chart components or Recharts for dashboard charts

### Backend

- Node.js LTS
- Express
- TypeScript with strict mode
- PostgreSQL
- Prisma ORM
- JWT
- bcrypt
- Zod
- cookie-parser
- cors
- helmet
- dotenv

Optional only when needed:

- Supertest and Vitest for focused API tests
- csv-parse for CSV import

### Database

Use PostgreSQL hosted on Supabase.

Use Prisma for:

- schema definition
- migrations
- queries
- relations
- seed data

Do not use Supabase Auth, Supabase client-side database access, Firebase, or another authentication provider. Authentication must be implemented by the Express backend using JWT and bcrypt.

---

## 5. Important Architecture Boundary

Next.js is the frontend framework only.

Do **not** implement business APIs using:

- Next.js Route Handlers
- `app/api`
- Pages Router API routes
- server actions that directly modify the database
- direct Prisma access from the frontend
- direct Supabase database queries from the frontend

All application data and authentication operations must go through the separate Express API.

The frontend may use Next.js layouts, pages, navigation, loading UI, and client components. It must call the Express backend through Axios.

---

## 6. Core Scope Decisions

Use the following decisions consistently unless the assignment explicitly requires otherwise.

### Authentication

- No public registration.
- Seed one Super Admin account.
- Login uses email and password.
- The backend signs a short-lived JWT.
- Store the JWT in an HTTP-only cookie.
- Logout clears the cookie.
- Add `GET /api/auth/me` so the frontend can restore the current session.
- Password reset, email verification, refresh-token rotation, and social login are out of scope.

### Employee and Account Model

Use one `Employee` database model for both:

- employee profile information
- login credentials
- role and authorization

Do not create separate User and Employee models unless there is a clear implementation need.

### Profile Image

For the core assignment, store:

```text
profileImageUrl: string | null
```

Provide an image URL field and avatar preview.

Do not add S3, Cloudinary, image transformation, or a custom upload pipeline unless all core requirements are already complete.

### Departments

Store department as a normalized string field on the employee record.

A separate Department table is not required for this assignment.

Department count means the number of distinct non-deleted department values.

### Delete Behavior

Prefer soft delete using `deletedAt`.

- Only Super Admin may delete an employee.
- Deleted employees must not appear in normal lists, counts, manager selectors, or the organization tree.
- Do not permanently delete records unless soft delete becomes a blocker.

### Employee Self-Editing

An Employee role may view their own profile and update only:

- phone
- profile image URL

Name, email, salary, department, designation, role, status, joining date, employee ID, and manager are controlled by HR or Super Admin.

---

## 7. Roles and Permissions

Roles must be represented with fixed enum values:

```text
SUPER_ADMIN
HR_MANAGER
EMPLOYEE
```

Never use arbitrary role strings.

### Permission Matrix

| Capability | Super Admin | HR Manager | Employee |
|---|---:|---:|---:|
| View dashboard metrics | Yes | Yes | No |
| View employee list | Yes | Yes | No |
| View any employee | Yes | Yes | No |
| View own profile | Yes | Yes | Yes |
| Create employee | Yes | Yes | No |
| Edit general employee fields | Yes | Yes | No |
| Delete employee | Yes | No | No |
| Assign or change manager | Yes | No | No |
| Assign Super Admin role | Yes | No | No |
| Assign HR Manager or Employee role | Yes | Yes | No |
| Edit own limited fields | Yes | Yes | Yes |
| View organization tree | Yes | Yes | No |
| View direct reportees | Yes | Yes | No |

Frontend permission checks are for user experience only.

Every permission must also be enforced by the Express backend.

---

## 8. Required Application Screens

### Public

- Login page

### Super Admin and HR Manager

- Dashboard
- Employee list
- Create employee
- Employee details
- Edit employee
- Organization hierarchy
- Direct reportees view or section

### Employee

- Own profile
- Edit own allowed fields

### Shared

- Unauthorized page or clear forbidden state
- Not-found state
- Loading state
- Error state

Do not build public landing pages, onboarding flows, audit dashboards, chat systems, notification centers, or unrelated modules.

---

## 9. Required Functional Behavior

### Dashboard

Display:

- total employees
- active employees
- inactive employees
- department count

Dashboard charts are a bonus. Metric cards are required.

### Employee Management

Support:

- create
- list
- view
- update
- soft delete
- search by name or email
- filter by department
- filter by role
- filter by status
- sort by name
- sort by joining date
- server-side pagination

### Organization Hierarchy

Support:

- assign a reporting manager
- clear a reporting manager
- display a nested reporting tree
- show direct reports
- prevent self-reporting
- prevent circular reporting
- exclude deleted employees from hierarchy results

Circular reporting validation must happen in the backend, not only in the UI.

---

## 10. API Conventions

Use the base path:

```text
/api
```

Use JSON for requests and responses except future optional file imports.

### Success Response

```json
{
  "success": true,
  "message": "Employee created successfully",
  "data": {}
}
```

### Paginated Success Response

```json
{
  "success": true,
  "message": "Employees fetched successfully",
  "data": [],
  "meta": {
    "page": 1,
    "limit": 10,
    "total": 42,
    "totalPages": 5
  }
}
```

### Error Response

```json
{
  "success": false,
  "message": "Validation failed",
  "errors": [
    {
      "field": "email",
      "message": "Enter a valid email address"
    }
  ]
}
```

Do not return stack traces, password hashes, JWT secrets, or internal database details.

### Status Codes

Use status codes consistently:

- `200` successful read or update
- `201` successful creation
- `204` optional for successful empty responses
- `400` malformed request
- `401` unauthenticated
- `403` authenticated but forbidden
- `404` resource not found
- `409` duplicate or hierarchy conflict
- `422` schema validation failure
- `500` unexpected server error

---

## 11. Required API Surface

Implement at least:

```text
POST   /api/auth/login
POST   /api/auth/logout
GET    /api/auth/me

GET    /api/dashboard/summary

GET    /api/employees
POST   /api/employees
GET    /api/employees/:id
PUT    /api/employees/:id
DELETE /api/employees/:id

GET    /api/organization/tree
GET    /api/employees/:id/reportees
PATCH  /api/employees/:id/manager
```

Optional:

```text
POST   /api/employees/import
```

Do not add GraphQL, tRPC, WebSockets, event queues, or microservices.

---

## 12. Validation Rules

Validate on both frontend and backend.

Backend validation is authoritative.

At minimum validate:

- required fields
- unique employee ID
- unique email
- valid email format
- reasonable phone format
- non-negative salary
- valid date
- valid role
- valid status
- manager existence
- manager is not the employee
- manager assignment does not create a cycle

Use Zod schemas instead of scattered manual validation.

Database unique constraints must back up application-level duplicate checks.

---

## 13. Status Values

Use fixed employee status values:

```text
ACTIVE
INACTIVE
```

Use a shadcn `Badge` to display status and role in the frontend.

Do not derive active status from login activity.

---

## 14. UI and UX Standards

Use shadcn/ui components before building custom primitives.

Common components should include:

- Button
- Input
- Label
- Form
- Card
- Table
- Badge
- Avatar
- Dialog
- AlertDialog
- DropdownMenu
- Select
- Sheet
- Skeleton
- Tooltip
- Breadcrumb
- Pagination controls
- Sonner toast

Use:

- skeletons when the shape of loading content is known
- button spinners for form submission or short mutations
- empty states when no records match
- confirmation dialogs before destructive actions
- disabled controls when the user lacks permission
- toasts for mutation results
- inline validation messages for form fields

Never use browser `alert`, `confirm`, or `prompt`.

---

## 15. Theme and Styling Rules

Use Tailwind CSS and shadcn semantic theme tokens.

Use classes such as:

```text
bg-background
text-foreground
bg-card
text-card-foreground
border-border
text-muted-foreground
bg-primary
text-primary-foreground
bg-destructive
text-destructive-foreground
```

Avoid hardcoded visual palette classes such as:

```text
bg-blue-500
text-red-600
border-green-400
```

Hardcoded colors are permitted only when an external brand asset or a genuinely non-theme data visualization requires it.

Do not use inline style objects for normal styling.

Keep spacing, radius, typography, and states consistent.

The UI must work on desktop and smaller screens. A responsive table may become a horizontally scrollable table or a compact card list on mobile.

---

## 16. State Management

Use TanStack Query for all server state:

- current authenticated user
- dashboard metrics
- employee lists
- employee details
- direct reportees
- organization tree

Use local React state for:

- dialog visibility
- temporary UI selections
- table controls before applying filters
- sidebar state

Do not add Redux, Zustand, MobX, or another global state library.

A small auth provider is allowed, but it should be backed by the `auth/me` query rather than duplicating server state.

---

## 17. Error Handling

Errors must be handled deliberately.

### Frontend

- normalize Axios errors
- show a useful page or component error state
- show field validation errors inline
- show mutation failures through Sonner
- redirect to login on expired unauthenticated sessions when appropriate
- show an unauthorized state for `403`
- never expose raw backend stack traces

### Backend

- use a central error-handling middleware
- use typed application errors
- catch and map Prisma errors
- map duplicate constraints to `409`
- map validation failures to `422`
- log unexpected errors
- do not wrap every controller in duplicated `try/catch` if the Express setup already forwards async errors correctly

---

## 18. Code Quality

Use strict TypeScript in both applications.

Rules:

- do not use `any`
- do not suppress TypeScript errors without a documented reason
- use descriptive names
- prefer small focused modules
- keep controllers and page components thin
- avoid large utility dumping grounds
- avoid premature generic abstractions
- avoid one-file implementations
- avoid duplicated permission logic
- remove dead code and generated demo content
- run formatting, linting, and type checking before submission

Comments should explain non-obvious decisions, especially hierarchy cycle prevention and RBAC restrictions. Do not comment obvious syntax.

---

## 19. Documentation Requirements

The root `README.md` should include:

1. project overview
2. feature list
3. tech stack
4. role and permission summary
5. architecture overview
6. repository structure
7. local setup
8. environment variables
9. database migration and seed steps
10. frontend and backend run commands
11. API summary
12. deployment links
13. screenshots or demo GIF
14. seeded login credentials for review
15. assumptions and scope decisions
16. bonus features implemented
17. known limitations

Do not include real secrets in the repository.

Provide:

```text
frontend/.env.example
backend/.env.example
```

---

## 20. Suggested Implementation Order

Complete work in this order:

1. repository and app setup
2. Prisma schema and migration
3. Super Admin seed
4. backend authentication
5. backend RBAC middleware
6. employee CRUD and validation
7. hierarchy assignment and circular prevention
8. dashboard summary
9. frontend authentication flow
10. dashboard UI
11. employee table and forms
12. role-aware actions
13. organization tree
14. loading, empty, error, and responsive states
15. README, screenshots, and deployment
16. bonus features only if time remains

Do not spend early time on charts, Docker, CSV import, or extensive tests while required functionality is incomplete.

---

## 21. Bonus Priority

Only implement bonus work after all required flows work end to end.

Recommended order:

1. pagination
2. soft delete
3. dark mode
4. one simple dashboard chart
5. focused unit or integration tests
6. CSV import
7. Docker

Pagination and soft delete may be treated as part of the main implementation because they are low-cost and improve review quality.

---

## 22. Definition of Done

The assignment is done when:

- a seeded Super Admin can log in
- authentication survives page refresh
- logout works
- protected pages reject unauthenticated users
- all three roles are enforced by the backend
- HR cannot delete an employee
- HR cannot assign Super Admin
- Employee cannot access the employee list or dashboard metrics
- Employee can update only allowed own fields
- employee CRUD works according to permissions
- search, filters, sorting, and pagination work
- dashboard counts are correct
- manager assignment works
- circular reporting is rejected
- organization tree and direct reports display correctly
- loading, empty, error, and success states are visible
- frontend and backend are deployed independently from the same repository
- README and environment examples are complete

---

## 23. Final Agent Rules

Before adding a dependency, confirm that it solves a real requirement.

Before creating an abstraction, confirm that it will be reused.

Do not replace working assignment code with a complex architecture merely to appear production-ready.

Prefer a smaller application where every required flow works over a larger application with unfinished features.

Do not silently change the defined stack, roles, API paths, response format, or permissions.
