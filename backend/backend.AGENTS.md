# AGENTS.md — Backend

## 1. Backend Scope

This folder contains the Express API for the Employee Management System.

The backend is responsible for:

- PostgreSQL access
- Prisma schema and migrations
- authentication
- JWT creation and verification
- password hashing
- HTTP-only auth cookie
- role-based authorization
- field-level authorization
- employee CRUD
- dashboard metrics
- search, filters, sorting, and pagination
- reporting hierarchy
- circular-reporting prevention
- request validation
- consistent API responses
- centralized error handling
- seed data

The backend is the authoritative security boundary.

Never trust role, employee ID, editable fields, salary, manager, or other permission-sensitive information supplied by the frontend.

---

## 2. Backend Stack

Use:

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

Optional:

- Vitest
- Supertest
- csv-parse

Do not use:

- NestJS
- GraphQL
- tRPC
- Supabase Auth
- Firebase Auth
- direct frontend database access
- multiple services or microservices
- Redis unless a real requirement appears

---

## 3. Suggested Backend Structure

Use a small layered structure:

```text
backend/
├── AGENTS.md
├── prisma/
│   ├── schema.prisma
│   ├── migrations/
│   └── seed.ts
├── src/
│   ├── app.ts
│   ├── server.ts
│   ├── config/
│   │   ├── env.ts
│   │   └── cors.ts
│   ├── controllers/
│   │   ├── auth.controller.ts
│   │   ├── dashboard.controller.ts
│   │   ├── employee.controller.ts
│   │   └── organization.controller.ts
│   ├── services/
│   │   ├── auth.service.ts
│   │   ├── dashboard.service.ts
│   │   ├── employee.service.ts
│   │   └── organization.service.ts
│   ├── routes/
│   │   ├── auth.routes.ts
│   │   ├── dashboard.routes.ts
│   │   ├── employee.routes.ts
│   │   └── organization.routes.ts
│   ├── middlewares/
│   │   ├── authenticate.ts
│   │   ├── authorize.ts
│   │   ├── validate.ts
│   │   ├── not-found.ts
│   │   └── error-handler.ts
│   ├── schemas/
│   │   ├── auth.schema.ts
│   │   ├── employee.schema.ts
│   │   └── organization.schema.ts
│   ├── utils/
│   │   ├── app-error.ts
│   │   ├── api-response.ts
│   │   ├── cookies.ts
│   │   ├── jwt.ts
│   │   └── pagination.ts
│   ├── types/
│   │   ├── auth.ts
│   │   ├── express.d.ts
│   │   └── api.ts
│   └── lib/
│       └── prisma.ts
├── tests/
├── .env.example
├── package.json
└── tsconfig.json
```

Do not create empty layers merely to match the diagram.

Controllers should translate HTTP input and output.

Services should contain business rules and Prisma operations.

Validation should happen before controller logic.

---

## 4. Prisma Data Model

Use one `Employee` model.

Recommended schema shape:

```prisma
enum EmployeeRole {
  SUPER_ADMIN
  HR_MANAGER
  EMPLOYEE
}

enum EmployeeStatus {
  ACTIVE
  INACTIVE
}

model Employee {
  id              String         @id @default(uuid())
  employeeId      String         @unique
  name            String
  email           String         @unique
  passwordHash    String
  phone           String
  department      String
  designation     String
  salary          Decimal        @db.Decimal(12, 2)
  joiningDate     DateTime
  status          EmployeeStatus @default(ACTIVE)
  role            EmployeeRole   @default(EMPLOYEE)
  profileImageUrl String?

  managerId       String?
  manager         Employee?      @relation("EmployeeManager", fields: [managerId], references: [id], onDelete: SetNull)
  reportees       Employee[]     @relation("EmployeeManager")

  createdAt       DateTime       @default(now())
  updatedAt       DateTime       @updatedAt
  deletedAt       DateTime?

  @@index([name])
  @@index([email])
  @@index([department])
  @@index([role])
  @@index([status])
  @@index([joiningDate])
  @@index([managerId])
}
```

Formatting may be adjusted to valid Prisma syntax.

### Data Rules

- `employeeId` is user-facing and unique.
- `id` is the internal UUID.
- `email` should be normalized to lowercase before storage.
- `passwordHash` must never be selected into normal API responses.
- `salary` must be non-negative.
- `managerId` is nullable.
- `deletedAt` implements soft delete.
- deleted employees are excluded by default.

Do not store manager name, department count, reportee count, or other derived values redundantly.

---

## 5. Decimal Serialization

Prisma `Decimal` values require deliberate response serialization.

Choose one consistent API representation:

```text
salary as a number
```

or:

```text
salary as a decimal string
```

For this assignment, returning a number is acceptable if salary values stay in a safe practical range.

Create one employee serializer rather than converting Decimal values ad hoc across controllers.

The serializer must also remove:

- passwordHash
- internal-only fields not needed by the frontend

---

## 6. Seed Data

Create a Prisma seed that inserts at least:

- one Super Admin
- one HR Manager
- several Employees
- multiple departments
- active and inactive records
- a small reporting hierarchy

Make the seed idempotent using upsert where practical.

Use environment variables for seeded Super Admin credentials:

```bash
SUPER_ADMIN_EMAIL=admin@example.com
SUPER_ADMIN_PASSWORD=ChangeMe123!
```

The seed must hash passwords using bcrypt.

The README may provide assignment/demo credentials, but never commit a real personal password.

---

## 7. Authentication Strategy

### Login

Endpoint:

```text
POST /api/auth/login
```

Request:

```json
{
  "email": "admin@example.com",
  "password": "ChangeMe123!"
}
```

Behavior:

1. validate request with Zod
2. normalize email
3. find a non-deleted employee
4. reject inactive employee accounts
5. compare password with bcrypt
6. create a signed JWT
7. set the HTTP-only auth cookie
8. return a safe current-user object

Use one generic invalid-credentials message so login does not reveal whether an email exists.

### JWT Payload

Keep the JWT payload minimal:

```ts
type JwtPayload = {
  sub: string;
  role: EmployeeRole;
};
```

Do not put salary, department, or a full employee object in the token.

The database remains authoritative for current status and role. The authentication middleware should load the current employee or otherwise ensure deleted/inactive accounts do not retain access.

### Cookie

Recommended cookie name:

```text
ems_token
```

Cookie settings:

- `httpOnly: true`
- `secure: true` in production
- `sameSite: "none"` in production when frontend and backend are on different sites
- a local-development-compatible SameSite setting
- finite `maxAge`
- `path: "/"`

When using cross-site cookies in production:

- Express CORS must use the exact frontend origin
- `credentials: true` must be enabled
- wildcard origins must not be used with credentials

### Logout

Endpoint:

```text
POST /api/auth/logout
```

Clear the cookie using matching cookie options.

A server-side token blacklist is not required for this assignment.

Use a reasonably short token expiration.

### Current User

Endpoint:

```text
GET /api/auth/me
```

Return a safe user payload:

```json
{
  "id": "uuid",
  "employeeId": "EMP-001",
  "name": "Admin User",
  "email": "admin@example.com",
  "phone": "...",
  "department": "Administration",
  "designation": "Super Admin",
  "status": "ACTIVE",
  "role": "SUPER_ADMIN",
  "profileImageUrl": null
}
```

Never return `passwordHash`.

---

## 8. Password Handling

Use bcrypt.

Rules:

- hash all passwords before storage
- never log passwords
- never return password hashes
- never accept a pre-hashed password
- apply a reasonable password minimum length
- use a cost factor appropriate for an assignment, such as 10–12
- require a password when creating an employee account
- do not return the initial password in API responses

Password reset and password-change flows are out of scope unless core work is complete.

---

## 9. Authentication Middleware

Create an `authenticate` middleware.

It should:

1. read the JWT from the HTTP-only cookie
2. verify signature and expiration
3. validate the payload shape
4. load the employee
5. reject missing, deleted, or inactive employees
6. attach a safe authenticated employee object to `req.user`

Unauthenticated requests return `401`.

Extend Express request typing in one declaration file.

Do not cast `req as any`.

---

## 10. Authorization Middleware

Create a reusable role middleware such as:

```ts
authorize("SUPER_ADMIN", "HR_MANAGER")
```

Use route-level authorization for broad permissions and service-level checks for resource or field-specific permissions.

Examples:

- dashboard routes → Super Admin or HR Manager
- employee list → Super Admin or HR Manager
- create employee → Super Admin or HR Manager
- delete employee → Super Admin only
- manager assignment → Super Admin only
- organization tree → Super Admin or HR Manager

Do not rely only on route-level role checks for self-editing or role assignment restrictions.

---

## 11. Field-Level Authorization

### Super Admin

May:

- create employees
- view all employees
- edit all employee fields
- soft delete employees
- assign any role
- assign or clear managers

Protect the last active Super Admin from being deleted, deactivated, or demoted if implementing that check is straightforward. This is recommended but not mandatory for the assignment.

### HR Manager

May:

- create employees
- view all non-deleted employees
- update general employee data
- assign `HR_MANAGER` or `EMPLOYEE`
- update status

May not:

- delete employees
- assign `SUPER_ADMIN`
- edit or demote an existing Super Admin
- assign or clear reporting managers
- modify password hashes directly

### Employee

May:

- view own profile
- update own phone
- update own profile image URL

May not:

- view employee list
- view another employee
- update restricted own fields
- alter role, status, salary, manager, or organization data

Do not pass request bodies directly to Prisma update methods.

Explicitly pick allowed fields based on the authenticated role and target record.

---

## 12. Request Validation

Use Zod schemas for:

- login
- employee creation
- employee update
- employee self-update
- employee list query parameters
- ID route parameters
- manager assignment

Create a reusable validation middleware that can validate:

- `body`
- `params`
- `query`

Return `422` with field-level errors.

### Suggested Validation

#### Employee ID

- trim whitespace
- required
- reasonable maximum length
- allow letters, numbers, hyphens, and underscores

#### Name

- trim whitespace
- required
- reasonable minimum and maximum length

#### Email

- valid email
- lowercase before querying or storing

#### Phone

- required for employee create
- accept a practical international phone format
- normalize spaces and separators if needed
- store consistently

#### Department and Designation

- trim
- required
- reasonable maximum lengths

#### Salary

- numeric
- non-negative
- finite
- suitable upper bound if desired

#### Joining Date

- valid ISO date input
- convert once in the service layer

#### Role and Status

- use exact enums

#### Profile Image URL

- nullable or empty
- valid URL when present

#### Password

- required on create
- minimum length
- never accepted on normal employee update unless a specific password endpoint is added

---

## 13. Employee List Endpoint

Endpoint:

```text
GET /api/employees
```

Allowed roles:

- Super Admin
- HR Manager

Suggested query parameters:

```text
page=1
limit=10
search=aditya
department=Engineering
role=EMPLOYEE
status=ACTIVE
sortBy=name
sortOrder=asc
```

Supported sorting:

```text
name
joiningDate
```

Supported order:

```text
asc
desc
```

### Query Behavior

- exclude soft-deleted employees
- search name and email case-insensitively
- apply optional filters
- use server-side pagination
- enforce a maximum page size
- return total count and total pages
- use an allowlist for sort fields
- never inject raw user input into SQL or `orderBy`

Suggested defaults:

```text
page=1
limit=10
sortBy=joiningDate
sortOrder=desc
```

Return only fields required for the list.

Do not select password hashes.

---

## 14. Employee Create Endpoint

Endpoint:

```text
POST /api/employees
```

Allowed roles:

- Super Admin
- HR Manager

Behavior:

1. validate input
2. enforce role assignment permission
3. normalize email
4. check duplicate employee ID and email
5. hash password
6. create employee
7. return safe serialized employee

If a manager field is included:

- accept it only for Super Admin
- validate manager existence
- reject deleted manager
- circular validation is not needed for a new employee with no reportees, but self-ID validation still applies if IDs are client-supplied

Prefer manager assignment through the dedicated manager endpoint to keep permissions clear.

---

## 15. Employee Detail Endpoint

Endpoint:

```text
GET /api/employees/:id
```

Access:

- Super Admin → any non-deleted employee
- HR Manager → any non-deleted employee
- Employee → own record only

Return useful relations:

- safe manager summary
- direct report count
- optional direct report summaries

Do not return deeply recursive hierarchy data from this endpoint.

---

## 16. Employee Update Endpoint

Endpoint:

```text
PUT /api/employees/:id
```

Use PUT because it is explicitly requested by the assignment, even if the update behaves like a partial update.

Clearly document this behavior.

### Update Rules

- validate route ID
- load target employee
- reject deleted target
- apply role-based field allowlist
- normalize email
- enforce email and employee ID uniqueness
- do not accept `passwordHash`
- do not accept arbitrary `managerId` through this endpoint
- handle role restrictions
- serialize response safely

Employee self-update must use a schema that allows only:

```text
phone
profileImageUrl
```

Do not validate restricted fields and then ignore them silently. Reject unexpected restricted fields or strip through an explicit allowlist and document the behavior.

---

## 17. Employee Delete Endpoint

Endpoint:

```text
DELETE /api/employees/:id
```

Allowed role:

- Super Admin only

Use soft delete:

```text
deletedAt = current timestamp
status = INACTIVE
```

Recommended related behavior:

- clear this employee as manager for active direct reportees, or explicitly reject deletion until reportees are reassigned
- choose one behavior and document it

Preferred assignment-friendly behavior:

1. soft delete employee
2. set `managerId = null` for their active direct reportees in the same transaction

Reject deleting the currently authenticated Super Admin account if it would make testing impossible.

Return a success message.

---

## 18. Dashboard Summary

Endpoint:

```text
GET /api/dashboard/summary
```

Allowed roles:

- Super Admin
- HR Manager

Return:

```json
{
  "totalEmployees": 25,
  "activeEmployees": 21,
  "inactiveEmployees": 4,
  "departmentCount": 5
}
```

All metrics must exclude soft-deleted employees.

Compute department count using distinct department values.

Do not calculate fake changes or historical trends.

---

## 19. Manager Assignment

Endpoint:

```text
PATCH /api/employees/:id/manager
```

Allowed role:

- Super Admin only

Request:

```json
{
  "managerId": "manager-uuid"
}
```

To clear manager:

```json
{
  "managerId": null
}
```

### Validation

Reject when:

- employee does not exist
- employee is deleted
- manager does not exist
- manager is deleted
- manager ID equals employee ID
- assignment creates a reporting cycle

Return `409` for hierarchy conflicts.

---

## 20. Circular Reporting Prevention

Circular validation is mandatory.

Example invalid hierarchy:

```text
A reports to B
B reports to C
Attempt: C reports to A
```

### Recommended Algorithm

When assigning `newManagerId` to `employeeId`:

1. if `newManagerId` is null, allow clearing
2. if both IDs are equal, reject
3. start at the proposed manager
4. repeatedly load the current manager's `managerId`
5. if the chain reaches `employeeId`, reject the assignment
6. if the chain reaches null, assignment is valid
7. track visited IDs as defensive protection against pre-existing corrupt cycles

Pseudocode:

```ts
async function assertNoReportingCycle(
  employeeId: string,
  newManagerId: string | null
): Promise<void> {
  if (!newManagerId) return;

  if (employeeId === newManagerId) {
    throw new ConflictError(
      "An employee cannot report to themselves."
    );
  }

  const visited = new Set<string>();
  let currentId: string | null = newManagerId;

  while (currentId) {
    if (currentId === employeeId) {
      throw new ConflictError(
        "This manager assignment would create a reporting cycle."
      );
    }

    if (visited.has(currentId)) {
      throw new ConflictError(
        "The existing reporting hierarchy contains a cycle."
      );
    }

    visited.add(currentId);

    const current = await prisma.employee.findFirst({
      where: {
        id: currentId,
        deletedAt: null,
      },
      select: {
        managerId: true,
      },
    });

    currentId = current?.managerId ?? null;
  }
}
```

The exact implementation may be optimized, but it must remain clear and correct.

Do not rely on the organization-tree builder to detect cycles after saving invalid data.

---

## 21. Direct Reportees Endpoint

Endpoint:

```text
GET /api/employees/:id/reportees
```

Allowed roles:

- Super Admin
- HR Manager

Return direct reports only, not every descendant, unless the response explicitly separates direct and nested reportees.

Exclude deleted employees.

Recommended fields:

- id
- employeeId
- name
- email
- designation
- department
- status
- role
- profileImageUrl

Support simple pagination only if needed. It is not required for this endpoint in a small assignment dataset.

---

## 22. Organization Tree Endpoint

Endpoint:

```text
GET /api/organization/tree
```

Allowed roles:

- Super Admin
- HR Manager

Recommended approach:

1. fetch all non-deleted employees in one query
2. select only hierarchy display fields
3. build a map keyed by employee ID
4. attach each employee node to its manager node
5. employees without a valid manager become roots
6. return an array of root nodes

Suggested response node:

```ts
type OrganizationNode = {
  id: string;
  employeeId: string;
  name: string;
  designation: string;
  department: string;
  role: EmployeeRole;
  status: EmployeeStatus;
  profileImageUrl: string | null;
  children: OrganizationNode[];
};
```

Do not recursively query the database once per employee.

Use defensive visited tracking while constructing or traversing the tree.

---

## 23. Prisma Query Rules

Use one Prisma client singleton.

Do not instantiate `PrismaClient` in every service.

Use:

- `select` to avoid leaking fields
- transactions for multi-record changes
- database unique constraints
- indexed query fields
- `findFirst` with `deletedAt: null` for active record access
- `count` and distinct queries for dashboard metrics

Avoid raw SQL unless Prisma cannot reasonably express the requirement.

Do not use `include: true` broadly when a smaller `select` is sufficient.

---

## 24. Consistent API Responses

Create helpers for success responses.

Example:

```ts
res.status(200).json({
  success: true,
  message: "Employee fetched successfully",
  data: employee,
});
```

Validation error:

```ts
res.status(422).json({
  success: false,
  message: "Validation failed",
  errors: [
    {
      field: "email",
      message: "Enter a valid email address",
    },
  ],
});
```

Unexpected errors should use a generic production message.

Do not return Prisma error objects directly.

---

## 25. Error Handling

Create an `AppError` or equivalent containing:

- status code
- public message
- optional error code
- optional field errors
- operational flag if useful

Use a final Express error middleware.

Handle at least:

- Zod validation errors
- invalid JWT
- expired JWT
- duplicate Prisma constraints
- missing records
- forbidden role actions
- hierarchy conflicts
- unexpected server errors

Log unexpected errors with enough context for debugging, but never log:

- raw passwords
- auth cookies
- JWT secrets
- password hashes
- full database URLs

Do not leave rejected promises unhandled.

---

## 26. Security Middleware

Configure:

- `helmet()`
- `express.json()` with a reasonable size limit
- `cookieParser()`
- CORS with exact allowed frontend origin
- credentials enabled
- environment validation at startup

Optional:

- basic login rate limiting if it can be added quickly

Do not add a broad security package collection that is not configured or understood.

---

## 27. CORS Configuration

Use:

```ts
cors({
  origin: env.CLIENT_URL,
  credentials: true,
});
```

For local development:

```text
CLIENT_URL=http://localhost:3000
```

For production:

```text
CLIENT_URL=https://your-frontend.vercel.app
```

Do not use:

```ts
origin: "*"
```

with cookie credentials.

If multiple explicit development origins are required, use an allowlist.

---

## 28. Environment Variables

Create `.env.example`:

```bash
NODE_ENV=development
PORT=5000

DATABASE_URL=postgresql://USER:PASSWORD@HOST:PORT/DATABASE

CLIENT_URL=http://localhost:3000

JWT_SECRET=replace-with-a-long-random-secret
JWT_EXPIRES_IN=8h
COOKIE_NAME=ems_token

SUPER_ADMIN_EMAIL=admin@example.com
SUPER_ADMIN_PASSWORD=ChangeMe123!
```

Validate required environment variables at startup.

Do not commit `.env`.

Do not hardcode deployment URLs.

---

## 29. Backend Scripts

Recommended scripts:

```json
{
  "scripts": {
    "dev": "tsx watch src/server.ts",
    "build": "tsc",
    "start": "node dist/server.js",
    "typecheck": "tsc --noEmit",
    "prisma:generate": "prisma generate",
    "prisma:migrate": "prisma migrate dev",
    "prisma:deploy": "prisma migrate deploy",
    "prisma:seed": "prisma db seed",
    "test": "vitest run"
  }
}
```

Adjust paths to the actual build configuration.

For Render:

- build should generate Prisma Client and compile TypeScript
- production migration should use `prisma migrate deploy`
- start should run compiled JavaScript

Do not run `prisma migrate dev` in production.

---

## 30. Controller and Service Responsibilities

### Controllers

May:

- read validated request data
- access `req.user`
- call a service
- select HTTP status
- send standardized responses

Should not:

- contain long Prisma queries
- hash passwords directly
- implement hierarchy loops
- duplicate role logic
- parse raw query parameters repeatedly

### Services

Should:

- enforce business permissions
- query Prisma
- normalize data
- hash passwords
- check duplicates
- execute transactions
- validate hierarchy rules
- return safe domain results

Do not create an additional repository layer unless it clearly reduces duplication. For this assignment, Prisma calls inside services are acceptable.

---

## 31. API Documentation

Document every endpoint in the root README or a focused file such as:

```text
backend/docs/api.md
```

For each endpoint include:

- method and path
- authentication requirement
- allowed roles
- request body
- query parameters
- success response
- common errors

OpenAPI/Swagger is optional.

Do not spend core assignment time generating a complete Swagger UI unless the application is already finished.

---

## 32. Backend Testing Scope

Tests are a bonus, but a few high-value tests demonstrate correctness.

Prioritize:

1. login success and failure
2. HR cannot delete
3. HR cannot assign Super Admin
4. Employee cannot update restricted own fields
5. circular manager assignment is rejected
6. dashboard excludes deleted records

Use a test database if practical.

Do not mock away the exact hierarchy logic being tested.

---

## 33. Backend Deployment

Deploy `backend/` as a Render web service from the same GitHub repository.

Configure:

```text
Root Directory: backend
```

Set all backend environment variables in Render.

Use Supabase PostgreSQL through `DATABASE_URL`.

Production requirements:

- build succeeds
- Prisma Client is generated
- migrations deploy
- server listens on `process.env.PORT`
- CORS allows the deployed frontend
- cookie options support HTTPS and the cross-site frontend
- health check or root response is helpful but optional

Suggested health endpoint:

```text
GET /api/health
```

Response:

```json
{
  "success": true,
  "message": "API is healthy"
}
```

Do not expose environment details in the health response.

---

## 34. Backend Definition of Done

The backend is complete when:

- Prisma migration succeeds
- seed creates working reviewer accounts
- login hashes and verifies passwords correctly
- JWT is set in an HTTP-only cookie
- logout clears the cookie
- `auth/me` restores current user
- inactive or deleted users cannot authenticate
- protected routes reject unauthenticated requests
- role middleware returns correct `403` responses
- HR cannot delete
- HR cannot assign Super Admin
- Employee can update only phone and profile image URL
- employee list supports search, filter, sort, and pagination
- duplicate email and employee ID are handled cleanly
- dashboard metrics are correct
- manager assignment supports null
- self-reporting and circular reporting are rejected
- direct reportees and organization tree are correct
- deleted records are excluded
- no response leaks password hashes
- CORS and cookies work with the deployed frontend
- build and type checking pass

---

## 35. Final Backend Agent Rules

Do not trust the frontend for authorization.

Do not pass arbitrary request objects into Prisma.

Do not return entire Prisma records when they contain sensitive fields.

Do not add complex infrastructure before required endpoints work.

Do not use Next.js server code as a second backend.

Do not silently weaken role rules to make the UI easier.

Keep the hierarchy implementation clear enough to explain in an interview.
