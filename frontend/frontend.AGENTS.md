<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

# AGENTS.md — Frontend

## 1. Frontend Scope

This folder contains the Next.js frontend for the Employee Management System.

The frontend is responsible for:

- pages and layouts
- login UI
- protected navigation
- role-aware user experience
- dashboard presentation
- employee forms and tables
- organization hierarchy presentation
- client-side validation
- server-state fetching and mutations
- responsive loading, empty, error, and success states

The frontend is **not** responsible for:

- database access
- password hashing
- JWT signing
- authoritative authorization
- hierarchy cycle validation
- business API implementation

All data operations must call the Express backend.

---

## 2. Frontend Stack

Use:

- Next.js App Router
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

Optional:

- `next-themes` for dark mode
- shadcn chart components or Recharts for one bonus chart

Do not use:

- Next.js API routes
- Route Handlers
- direct Prisma imports
- direct Supabase database calls
- Redux or another global state library
- a second component library such as Material UI
- server actions for backend business mutations

---

## 3. Suggested Frontend Structure

Keep the structure clear and assignment-sized:

```text
frontend/
├── AGENTS.md
├── app/
│   ├── (auth)/
│   │   └── login/
│   │       └── page.tsx
│   ├── (dashboard)/
│   │   ├── layout.tsx
│   │   ├── dashboard/
│   │   │   └── page.tsx
│   │   ├── employees/
│   │   │   ├── page.tsx
│   │   │   ├── new/
│   │   │   │   └── page.tsx
│   │   │   └── [id]/
│   │   │       ├── page.tsx
│   │   │       └── edit/
│   │   │           └── page.tsx
│   │   ├── organization/
│   │   │   └── page.tsx
│   │   └── profile/
│   │       └── page.tsx
│   ├── unauthorized/
│   │   └── page.tsx
│   ├── globals.css
│   ├── layout.tsx
│   ├── loading.tsx
│   └── not-found.tsx
├── components/
│   ├── auth/
│   ├── dashboard/
│   ├── employees/
│   ├── organization/
│   ├── layout/
│   ├── shared/
│   └── ui/
├── hooks/
│   ├── queries/
│   └── mutations/
├── lib/
│   ├── api-client.ts
│   ├── api-error.ts
│   ├── permissions.ts
│   ├── query-client.ts
│   └── utils.ts
├── providers/
│   ├── app-providers.tsx
│   └── theme-provider.tsx
├── schemas/
├── services/
├── types/
├── .env.example
└── package.json
```

This is a guide, not a requirement to create empty folders.

Do not create index-barrel files everywhere. Use direct imports unless a barrel materially improves readability.

---

## 4. Rendering and Component Strategy

Use server components for static layout when convenient.

Use client components for:

- TanStack Query hooks
- forms
- tables with interactions
- dialogs
- dropdowns
- browser-only behavior
- theme controls

Do not force all pages to be server components.

Do not fetch protected EMS data directly from server components unless cookie forwarding and error behavior are intentionally implemented. For this assignment, using client-side TanStack Query after authentication is acceptable and simpler.

Add `"use client"` only to files that require it.

---

## 5. Routing and Access Behavior

### Public Route

```text
/login
```

### Admin and HR Routes

```text
/dashboard
/employees
/employees/new
/employees/[id]
/employees/[id]/edit
/organization
```

### Employee Route

```text
/profile
```

### Navigation Rules

- Unauthenticated users visiting protected routes should be sent to `/login`.
- Authenticated Super Admin and HR Manager users may access dashboard, employee, and organization routes.
- Authenticated Employee users should be sent to `/profile` when they attempt to open admin routes.
- Authenticated users opening `/login` should be redirected to their correct default route.
- Show `/unauthorized` or an equivalent forbidden state for a `403`.

Frontend routing checks improve UX but are not a security boundary.

Prefer an auth-guard component or protected layout backed by the `auth/me` query. Do not duplicate backend JWT verification or role authorization inside Next.js middleware.

---

## 6. Authentication Client

Use cookie-based authentication.

The Axios client must use:

```ts
withCredentials: true
```

Do not store the JWT in:

- localStorage
- sessionStorage
- a readable JavaScript cookie
- React state

The frontend should restore authentication through:

```text
GET /api/auth/me
```

The current-user query is the single source of truth for session state.

Suggested auth query key:

```ts
["auth", "me"]
```

On successful login:

1. invalidate or refetch `["auth", "me"]`
2. show a success toast
3. route by role:
   - Super Admin → `/dashboard`
   - HR Manager → `/dashboard`
   - Employee → `/profile`

On logout:

1. call the logout endpoint
2. clear auth-related cached queries
3. redirect to `/login`
4. show a success toast

Do not display a protected page while auth status is unresolved. Show a full-page skeleton or centered loading state.

---

## 7. Axios API Client

Create one configured Axios instance in:

```text
lib/api-client.ts
```

It should define:

- `baseURL` from `NEXT_PUBLIC_API_URL`
- `withCredentials: true`
- JSON headers where appropriate
- a reasonable timeout
- response error normalization

Do not call `axios.get()` or `axios.post()` directly from page or component files.

Create service modules such as:

```text
services/auth.service.ts
services/dashboard.service.ts
services/employees.service.ts
services/organization.service.ts
```

Services should contain HTTP details.

Query hooks should call services.

Components should call hooks.

Preferred flow:

```text
Component → query/mutation hook → service → Axios client → Express API
```

### Error Interceptor

The response interceptor may:

- normalize the backend error shape
- identify `401`, `403`, `409`, and validation errors
- avoid showing duplicate global toasts
- clear stale auth state on `401`

Do not make the interceptor responsible for every toast. Components or mutation hooks should show context-specific messages.

---

## 8. API Error Type

Create a normalized frontend error type:

```ts
type ApiFieldError = {
  field?: string;
  message: string;
};

type ApiError = {
  status: number;
  message: string;
  errors?: ApiFieldError[];
};
```

Provide a helper such as:

```ts
getApiError(error: unknown): ApiError
```

Never access deeply nested Axios error fields throughout the UI.

Map backend field errors to React Hook Form using `setError` when possible.

---

## 9. TanStack Query Rules

Use TanStack Query for all backend state.

### Query Key Factory

Define stable keys, for example:

```ts
export const queryKeys = {
  auth: {
    me: ["auth", "me"] as const,
  },
  dashboard: {
    summary: ["dashboard", "summary"] as const,
  },
  employees: {
    all: ["employees"] as const,
    list: (params: EmployeeListParams) =>
      ["employees", "list", params] as const,
    detail: (id: string) =>
      ["employees", "detail", id] as const,
    reportees: (id: string) =>
      ["employees", "reportees", id] as const,
  },
  organization: {
    tree: ["organization", "tree"] as const,
  },
};
```

Do not use vague keys such as `["data"]`.

### Query Defaults

Configure sensible defaults:

- retry normal GET requests once
- do not retry authentication failures
- use a short `staleTime` for employee lists and dashboard data
- avoid refetching continuously
- preserve previous list data during pagination when useful

Suggested starting point:

```ts
staleTime: 30_000
retry: 1
refetchOnWindowFocus: false
```

These values may be adjusted per query.

### Mutations

Use `useMutation` for:

- login
- logout
- create employee
- update employee
- delete employee
- update own profile
- assign manager

After a successful employee mutation, invalidate only affected keys.

Examples:

- create employee:
  - employee lists
  - dashboard summary
  - organization tree when a manager was assigned
- update employee:
  - employee detail
  - employee lists
  - dashboard summary when status or department changed
  - organization tree when relevant
- delete employee:
  - employee lists
  - dashboard summary
  - organization tree
- assign manager:
  - employee detail
  - organization tree
  - old and new manager reportee queries

Prefer invalidation over manual refetch calls.

Do not copy fetched server data into long-lived component state unless a form is intentionally initialized from it.

---

## 10. TypeScript API Types

Define explicit types for:

- authenticated user
- employee
- employee role
- employee status
- dashboard summary
- employee list query parameters
- paginated metadata
- organization tree node
- API success and error responses
- create and update request bodies

Use string unions or enums that exactly match backend values:

```ts
type EmployeeRole =
  | "SUPER_ADMIN"
  | "HR_MANAGER"
  | "EMPLOYEE";

type EmployeeStatus =
  | "ACTIVE"
  | "INACTIVE";
```

Do not invent frontend-only role values.

Do not use `any`.

---

## 11. Forms

Use React Hook Form with Zod schemas.

Required forms:

- login
- create employee
- edit employee
- employee self-profile edit
- assign reporting manager

Place reusable schemas in `schemas/`.

Use `zodResolver`.

### Form Behavior

Every form must:

- have labels
- show inline validation messages
- mark required fields
- use appropriate input types
- disable submission while pending
- show a spinner inside the submit button while pending
- prevent duplicate submissions
- reset or navigate appropriately after success
- preserve useful entered values after a server error
- map backend field errors when available

Do not validate forms using only HTML `required`.

Backend validation remains authoritative.

### Employee Form Fields

Core employee fields:

- employee ID
- name
- email
- phone
- department
- designation
- salary
- joining date
- status
- role
- profile image URL

Manager assignment may be included in the Super Admin form or handled in a separate manager dialog. A separate dialog is preferable because it makes the permission boundary clear.

### Role-Aware Form Fields

For HR Manager:

- do not offer `SUPER_ADMIN` in the role selector
- do not render manager-assignment controls
- do not render delete actions

For Employee self-edit:

- render only phone and profile image URL
- do not merely disable restricted fields in a general employee form

---

## 12. shadcn/ui Component Usage

Prefer shadcn/ui components for all common interactions.

### Required or Strongly Preferred

- `Button`
- `Input`
- `Label`
- `Form`
- `Card`
- `Table`
- `Badge`
- `Avatar`
- `Dialog`
- `AlertDialog`
- `DropdownMenu`
- `Select`
- `Sheet`
- `Skeleton`
- `Tooltip`
- `Separator`
- `Breadcrumb`
- `ScrollArea`
- `Sonner`
- pagination controls

Use a custom component only when shadcn does not provide the required composition.

Do not install a second full UI library.

---

## 13. Loading States

Use skeletons when the result shape is known.

Examples:

- dashboard card skeletons
- employee table row skeletons
- employee profile card skeletons
- organization tree node skeletons

Use a spinner for:

- login button
- create/update submit button
- delete confirmation button
- manager assignment button
- very short indeterminate actions

Do not replace an entire content layout with a single spinner when skeletons would preserve visual stability.

---

## 14. Toasts

Use Sonner for:

- login success or failure
- logout
- employee created
- employee updated
- employee deleted
- manager assigned
- failed API actions
- CSV import result if implemented

Do not show a toast for every successful GET request.

Do not show both an interceptor toast and a component toast for the same error.

Use concise, actionable messages.

---

## 15. Badges and Display Formatting

Use `Badge` for:

- ACTIVE / INACTIVE
- SUPER_ADMIN / HR_MANAGER / EMPLOYEE

Create centralized formatting helpers:

```text
SUPER_ADMIN → Super Admin
HR_MANAGER → HR Manager
EMPLOYEE → Employee
```

Format:

- salary using `Intl.NumberFormat`
- dates using date-fns
- missing values as an em dash or clear placeholder
- avatar fallback using initials

Do not display raw enum values to users.

---

## 16. Theme Rules

Use shadcn CSS-variable theming.

Use semantic classes:

```text
bg-background
text-foreground
bg-card
text-card-foreground
border-border
text-muted-foreground
bg-muted
text-muted-foreground
bg-primary
text-primary-foreground
bg-destructive
text-destructive-foreground
```

Avoid hardcoded palette utilities:

```text
bg-blue-500
text-green-600
border-red-500
```

For positive or warning states, add semantic CSS variables if needed rather than scattering fixed colors.

Configure `components.json` to use CSS variables.

Dark mode is optional until required functionality is complete. The component styling must still be theme-compatible from the start.

---

## 17. Layout

Build one authenticated application shell containing:

- responsive sidebar
- mobile navigation sheet
- page title or breadcrumb
- current user menu
- role label
- logout action
- main content area

Navigation items must be role-aware.

### Super Admin and HR Manager Navigation

- Dashboard
- Employees
- Organization
- Profile

### Employee Navigation

- My Profile

Do not display links that will always lead to forbidden pages.

---

## 18. Dashboard Page

Display four metric cards:

- Total Employees
- Active Employees
- Inactive Employees
- Departments

Each card should contain:

- clear label
- count
- relevant Lucide icon
- loading skeleton
- error fallback

A single simple chart is optional after core completion.

Do not create fake percentage trends unless the backend provides real historical data.

---

## 19. Employee List Page

The employee list should support:

- debounced search by name or email
- department filter
- role filter
- status filter
- sort by name
- sort by joining date
- server-side pagination
- clear filters
- create employee action for allowed roles
- row actions based on permission

Recommended table columns:

- employee
- employee ID
- department
- designation
- role
- status
- joining date
- actions

Salary may be shown on the detail page instead of the main table if width is limited.

### URL State

Keeping search, filter, sort, and page values in URL search parameters is recommended but not mandatory.

Do not refetch on every uncontrolled keystroke. Debounce search input or apply it on submit.

### Empty States

Differentiate:

- no employees exist
- no employees match the current filters
- request failed

Provide a clear filter-reset action when appropriate.

---

## 20. Employee Detail Page

Show:

- profile image or initials
- name
- employee ID
- email
- phone
- department
- designation
- salary
- joining date
- status
- role
- reporting manager
- direct report count or list
- edit action when allowed
- delete action only for Super Admin
- manager assignment action only for Super Admin

Use cards and sections rather than one oversized table.

---

## 21. Organization Tree

Render a readable nested tree using the API-provided hierarchy.

Each node should show:

- avatar or initials
- employee name
- designation
- department
- role badge when useful
- direct report count when available

Use indentation and connecting visual structure without adding a heavy graph library.

Collapsible nodes are optional.

The backend should return already structured tree data. Do not implement cycle detection in the frontend.

Provide a useful empty state when no hierarchy exists.

---

## 22. Manager Assignment UI

Only Super Admin should see this control.

Use a dialog or select workflow.

Manager options should:

- exclude the employee being edited
- exclude deleted employees
- use a searchable selector if the list is large
- include a “No reporting manager” option
- show name, employee ID, and designation

The frontend may filter obviously invalid choices, but the backend must still perform complete circular validation.

When the backend rejects a cycle, show its message clearly.

---

## 23. Responsive Design

Required behavior:

- sidebar becomes a sheet or drawer on small screens
- metric cards wrap into one or two columns
- forms remain usable without horizontal scrolling
- tables use horizontal scrolling or a compact mobile presentation
- dialogs fit smaller viewports
- action buttons remain reachable
- text does not overflow cards

Test at common mobile, tablet, and desktop widths.

Do not optimize only for a wide desktop screenshot.

---

## 24. Accessibility

At minimum:

- use semantic buttons and links
- provide visible labels
- provide alt text for meaningful images
- ensure dialogs have titles and descriptions
- use keyboard-accessible shadcn controls
- preserve focus indicators
- provide sufficient contrast through theme tokens
- do not rely only on color to indicate status
- associate validation messages with inputs

Do not add clickable `div` elements when a button is appropriate.

---

## 25. Frontend Environment Variables

Create `.env.example` containing:

```bash
NEXT_PUBLIC_API_URL=http://localhost:5000/api
```

Use one consistent convention for whether `NEXT_PUBLIC_API_URL` includes `/api`.

Recommended:

```text
NEXT_PUBLIC_API_URL=https://backend.example.com/api
```

Service paths should then be relative, such as `/employees`.

Do not commit `.env.local`.

---

## 26. Frontend Quality Checks

Before submission, run:

```bash
npm run lint
npm run typecheck
npm run build
```

Add a `typecheck` script if one does not exist:

```json
{
  "scripts": {
    "typecheck": "tsc --noEmit"
  }
}
```

Resolve warnings that affect correctness or accessibility.

Do not leave default Next.js starter content or unused shadcn examples.

---

## 27. Frontend Testing Scope

Testing is a bonus.

If time remains, prioritize:

1. permission helper tests
2. employee form schema tests
3. one employee-list interaction test
4. one login flow test

Do not spend hours building a broad frontend test suite while core functionality is unfinished.

---

## 28. Frontend Definition of Done

The frontend is complete when:

- login and logout work through the Express backend
- auth restores after refresh
- role-specific redirects work
- protected pages do not flash sensitive content
- dashboard metrics render with skeletons and error states
- employee list search, filters, sort, and pagination work
- create and edit forms validate correctly
- actions are hidden or disabled according to role
- Employee self-edit renders only permitted fields
- delete uses a confirmation dialog
- manager assignment is Super Admin-only
- circular-assignment errors are shown clearly
- organization tree and reportees render
- mobile navigation and responsive layouts work
- mutation feedback uses Sonner
- styling uses semantic CSS-variable classes
- build, lint, and type checking pass

---

## 29. Final Frontend Agent Rules

Do not call the API directly inside visual components.

Do not store server data in Redux or duplicate it in context.

Do not use Next.js backend features to bypass the Express API.

Do not hardcode permission checks independently across many components. Centralize permission helpers.

Do not hide loading and error states.

Do not add visual complexity that delays completion of required workflows.
