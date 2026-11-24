# Design — Employee Profile View

| | |
|---|---|
| **Unit** | AXW-C001-U004 (employee-profile-view) |
| **Capability** | AXW-C001 — Features |
| **Kind** | api+ui |
| **Inputs** | intent.md (d7ab30d) · requirements.md (6b7862f, approved by Shiny Priscilla J, 2026-11-24) |
| **Status** | Approved by Shiny Priscilla J, 2026-11-24 (design and proposed stack D1) |
| **Author** | @architect (architect-unit-design) |

Everything marked **Proposed** is a draft decision for the accountable human to confirm.
Everything marked **UNRESOLVED** is not decided and must not be decided in code.

## 1. Approach

**Decision (proposed):** one read-only API operation, `getEmployeeProfile` (`GET /api/v1/employees/{employeeId}`), builds the whole profile on the server.
It reads the subject, the immediate manager and the active direct reports as one consistent snapshot.
It applies the permitted-set rule to the subject and to each related employee, and returns either the complete profile or a single uniform refusal.
The UI renders the response and adds no permission logic of its own.

**Rejected alternative:** the UI composes the profile from several calls (subject, manager, reports) and decides which related employees to link.
Rejected because:
- the browser would receive data about employees outside the viewer's permitted set (breaks R14, R17–R19 and OUT-7);
- partial failures would be possible (breaks A3, R24);
- several round trips put R16 (2 s at p95) at risk.

**Requirements issue raised with the BA (RQ1, non-blocking):** intent IN-8 and OUT-4 say the profile consumes an *existing* `GET /employees/{id}` service.
The user confirmed on 2026-11-24 that the API repo is new and ours to create, so no such service exists.
This design therefore builds `GET /api/v1/employees/{employeeId}` in this unit.
OUT-4 ("Building or changing the `GET /employees/{id}` service") needs amending by `ba-unit-requirements`; no R-ID changes.

## 2. Components & flows

### Components

| # | Component | Responsibility | R-IDs | Implementation |
|---|---|---|---|---|
| C1 | Profile endpoint | Accepts the request, gets the Viewer, calls C2, maps the outcome to 200 / 404 / 503 with `Cache-Control: no-store` | R1, R16, R17–R27 | API · `EmployeeProfileController.getEmployeeProfile()` · `src/employees/profile/employee-profile.controller.ts` (new) |
| C2 | Profile assembler | Orchestrates C3 and C4; builds the response; strips inactive related employees; sorts direct reports; blanks `employeeId` on related employees that cannot be opened | R2–R8, R13, R14, R20 | API · `EmployeeProfileService.getProfile()` · `src/employees/profile/employee-profile.service.ts` (new) |
| C3 | Permitted-set policy | Pure function: may this viewer open this employee? | R9–R12, R13, R14, R17–R19, R21, R22, R25–R27 | API · `PermittedSetPolicy.canOpen()` · `src/employees/profile/permitted-set.policy.ts` (new) |
| C4 | Profile snapshot reader | Reads the viewer record, the subject, the manager and the direct reports in one read-only transaction; times out at 1,500 ms | R2–R5, R16, R23, R24 | API · `EmployeeProfileRepository.loadSnapshot()` · `src/employees/profile/employee-profile.repository.ts` (new) |
| C5 | Viewer resolver | Supplies `Viewer { employeeId: string \| null; isHr: boolean }` to the controller. **Sign-in mechanism UNRESOLVED** (Q-D2) | R9–R12, R19 | API · `AuthGuard`, `@CurrentViewer()` · `src/common/auth/auth.guard.ts`, `src/common/auth/current-viewer.decorator.ts` (new) |
| C6 | Error envelope filter | Turns every error into `{ error: { code, message, requestId } }` | R17–R27 | API · `ApiErrorFilter` · `src/common/errors/api-error.filter.ts` (new) |
| C7 | Profile page | Route `/employees/:employeeId`; fetches the profile; renders the populated, loading and outcome states | R1–R27 | UI · `EmployeeProfilePage` · `src/features/employee-profile/EmployeeProfilePage.tsx` (new) |
| C8 | Back to Employee List | Returns to the list URL the user came from, keeping search, filters and page | R15 | UI · `BackToListLink` · `src/features/employee-profile/components/BackToListLink.tsx` (new) |
| C9 | List-row link (dependency on U003) | Each Employee List row's name opens `/employees/{employeeId}` and passes the list URL (with its query string) as router state `from` | R1, R15 | UI · U003's list row component — **not yet designed** (Q-D3) |

### Permitted-set rule (C3)
Source: requirements §2 "Permitted set", A2.

```
canOpen(viewer, viewerRecord, target):
  1. target is missing, or target.status ≠ ACTIVE           → false   (R21, R22, R25, R27)
  2. viewerRecord exists and viewerRecord.status ≠ ACTIVE   → false   (R26, A2)
  3. viewer.isHr                                            → true    (R9, R12)
  4. viewerRecord is missing                                → false   (R19, A2)
  5. target.employeeId = viewerRecord.employeeId            → true    (R10, R11)
  6. target.managerEmployeeId = viewerRecord.employeeId     → true    (R10)
  7. otherwise                                              → false   (R17, R18)
```

Manager status is worked out from the reporting line (rule 6), not from a role claim.
So the same rule covers employees, managers and users with both roles (R12).

### Flow F1 — open a profile (R1–R16, R20)

1. The user selects an employee in the Employee List (C9). The UI goes to `/employees/E10234` with state `from = "/employees?q=arj&department=Finance&page=2"`.
2. C7 shows the loading state and calls `getEmployeeProfile("E10234")`.
3. C5 resolves the Viewer.
4. C4 opens a read-only, repeatable-read transaction and reads: the viewer record (if `viewer.employeeId` is set), the subject, the subject's manager, and the subject's direct reports whose status is ACTIVE.
5. C3 checks `canOpen(subject)`. If false → F2.
6. C2 builds the response:
   - The manager is set to null if it is missing or inactive (R6, R7).
   - Direct reports are sorted by name using a case-insensitive `en` collator, then by employee ID (R4).
   - Each related employee's `employeeId` is kept only if `canOpen(related)` is true; otherwise it is null (R13, R14).
7. C1 returns 200 with `Cache-Control: no-store`.
8. C7 renders the profile and moves focus to the `<h1>` (the employee's name).

| Step fails | What the caller / user observes |
|---|---|
| 1 — list row has no `from` state (direct link, new tab) | The profile still opens; "Back to Employee List" goes to `/employees` with the list's defaults |
| 3 — not signed in | 401 `unauthenticated` (baseline; sign-in UNRESOLVED, Q-D2) |
| 4 — database down, or no answer within 1,500 ms | 503 `employee_details_unavailable`, `Retry-After: 5`; nothing partial is returned (R24, A3). The UI shows the recoverable error with "Try again" |
| 4 — the subject does not exist | F2 (R21, R22) |
| 5 — not permitted | F2 |
| 6 — unexpected fault | 500 `internal_error`; the UI shows the recoverable error state |
| 8 — browser offline when loading | The UI shows the offline message with "Try again" and retries automatically when back online |

### Flow F2 — uniform refusal (R17–R19, R21, R22, R25–R27)

1. C3 returns false, or the subject is missing, or the ID is malformed or longer than 64 characters (checked before any database read).
2. C1 returns 404 with the body `{ code: "profile_not_available", message: "Profile not available" }`. The body and headers are identical in every case. The only difference is `requestId`.
3. C7 shows the terminal "Profile not available" state, moves focus to its heading, and offers "Back to Employee List".

| Step fails | What the caller / user observes |
|---|---|
| 1 — the database is down while checking | 503 `employee_details_unavailable`, never 404. "Didn't answer" is not "said no" (R24) |
| 2 — timing differs between "unknown" and "not permitted" | Accepted risk (Decisions D8); the response body never differs |

### Flow F3 — open a related employee (R13, R27)

1. The user selects the manager or a direct report link (only rendered when `employeeId` is not null).
2. The UI goes to `/employees/{thatId}`, carrying the same `from` state forward; then F1 runs from step 2.

| Step fails | What the caller / user observes |
|---|---|
| 2 — the employee became inactive or left the viewer's set after the page loaded | F2: "Profile not available" (R27) |

### Flow F4 — back to the list (R15)

1. The user selects "Back to Employee List".
2. C8 navigates to `location.state.from` if it is a same-origin path starting with `/employees`; otherwise it navigates to `/employees`.
3. The list (U003) restores the search, filters and page from the URL query string.

| Step fails | What the caller / user observes |
|---|---|
| 2 — `from` missing or not a list path | The list opens with its defaults (name A–Z, page 1). No open redirect is possible |
| 3 — the list is unavailable | U003's `employee_list_unavailable` state (owned by U003) |

## 3. API

### Conventions (proposed)
- **Base path:** `/api/v1`. **Host: UNRESOLVED** (owner Shiny Priscilla J).
- **Authentication:** bearer scheme `viewerAuth`. The mechanism is **UNRESOLVED** (Q-D2). The API needs only `Viewer { employeeId: string | null, isHr: boolean }`.
- **Error envelope:** `{ "error": { "code": string, "message": string, "requestId": uuid } }`. `code` is contract; clients never branch on `message`.
- **Caching:** every response is `Cache-Control: no-store`, because the content depends on the viewer.
- **Pagination:** none. The single resource embeds `directReports` unpaged (Q5). This deviates from the "cursor pagination on every list" standard (Decision D6).
- **Idempotency:** not applicable. A GET with no side effects is safe to repeat (requirements: repetition N/A).

### Endpoints

| Operation | Method & path | Purpose | R-IDs |
|---|---|---|---|
| `getEmployeeProfile` | `GET /api/v1/employees/{employeeId}` | Profile of one employee in the viewer's permitted set | R1–R14, R16–R27 |

No write operations. Employee data is owned by U002; this unit exposes nothing writable (OUT-6).

#### `getEmployeeProfile`

**Request**

| Name | In | Type | Required | Rule (§2 field) |
|---|---|---|---|---|
| `employeeId` | path | string, 1+ characters | yes | Profile subject. Any value that does not identify an active employee in the viewer's permitted set → 404 `profile_not_available`, including malformed values and values over 64 characters (never 400) (R21, R22) |

**Response 200 — `EmployeeProfile`**

| Field | Type | Null? | Rule (§2 field) |
|---|---|---|---|
| `employeeId` | string | no | Directory information |
| `name` | string | no | Directory information |
| `designation` | string | yes | Same value as the list's "role" (Q1); null → UI "Not assigned" (A4) |
| `department` | string | yes | Null → UI "Not assigned" (A4) |
| `workEmail` | string (email) | no | Directory information (Q1) |
| `workPhone` | string | yes | Null → UI "Not recorded" (A4) |
| `manager` | `RelatedEmployee` | yes | Null when there is no manager or the manager is inactive (Q3, R6, R7) |
| `directReports` | `RelatedEmployee[]` | no (may be empty) | Active only; name A–Z, then employee ID; not paged (Q5, R4, R5, R8) |

`RelatedEmployee`: `employeeId` (string, **null when not openable**, Q2), `name` (string), `designation` (string, nullable).
No other fields exist: no salary, bank or personal information (OUT-1), and no manager's manager or indirect reports (OUT-2).

**Errors**

| HTTP | `code` | When | R-IDs |
|---|---|---|---|
| 404 | `profile_not_available` | Subject outside the permitted set, inactive, unknown or malformed; viewer has no permitted set | R17, R18, R19, R21, R22, R25, R26, R27 |
| 503 | `employee_details_unavailable` | Employee records unreadable or the read timed out (1,500 ms); `Retry-After: 5` | R24 |
| 401 | `unauthenticated` | Not signed in (project baseline; mechanism UNRESOLVED) | — (baseline) |
| 500 | `internal_error` | Unexpected fault (project baseline error model) | — (baseline) |

## 4. Data

**Datastore (proposed):** PostgreSQL 16, accessed through Prisma ORM 5.22.
Prisma owns the schema and migrations, so this unit writes no SQL; the entity model below is the storage contract (contracts.md, "ORM owns the schema").

**Ownership:** this unit **owns no entity**.
It reads `Employee`, whose data is maintained by U002 (requirements §2; U003 Q3).
Neither U002 nor U003 has a design yet, so **who defines the `Employee` model is UNRESOLVED** (Q-D1, owner Shiny Priscilla J).
The fields below are what this unit needs; the model's owner must provide them as stated.

### Entity read: `Employee` (Prisma model, `prisma/schema.prisma`)

| Field | Neutral type | Null? | Guarantee needed by this unit |
|---|---|---|---|
| `employeeId` | string | no | Identity; unique (primary key) |
| `fullName` | string | no | — |
| `designation` | string | yes | Same value the Employee List shows as "role" (Q1) |
| `department` | string | yes | — |
| `workEmail` | string | no | — |
| `workPhone` | string | yes | — |
| `managerEmployeeId` | string → `Employee.employeeId` | yes | Self-reference; an employee has at most one immediate manager |
| `status` | enum `ACTIVE` \| `INACTIVE` | no | Closed set |
| `createdAt`, `updatedAt` | timestamp (UTC) | no | Standard (owner's model) |

**Access patterns this unit relies on:**
1. Read one employee by `employeeId` (primary key): the subject, the viewer and the manager.
2. Read the active direct reports of one employee: `WHERE managerEmployeeId = :id AND status = 'ACTIVE'`. This needs an **index on `(managerEmployeeId, status)`**; the model owner must create it (R16).

**Retention and deletion:** owned by the project baseline and the model owner; this unit stores nothing.

### State & idempotency
- **State machine:** none. This unit owns no lifecycle. It reads `Employee.status` and treats only `ACTIVE` as visible.
- **Idempotency walk:** there is no effect to repeat. First attempt, client retry, double click and repeated "Try again" each perform only reads and return the current snapshot. A retry after a 503 simply reads again.
- **Consistency:** the four reads in C4 run in one `REPEATABLE READ` read-only transaction. So a profile never mixes before and after states of the same change (for example, a report counted under two managers).

### Concurrency matrix

| At the same time | Outcome | Enforced by |
|---|---|---|
| U002 changes Anil's manager or status while Ravi's profile is being read | The read sees either the old or the new state, never a mix | Store (repeatable-read snapshot) |
| U002 makes Anil inactive after Ravi's profile is shown | The shown profile stays as it is (no live update); the next open or reload omits Anil (R23) | UI: no polling, no refetch on focus |
| The subject becomes inactive after the profile is shown; the user reloads | 404 `profile_not_available` (R23, R25) | Application (C3 on every request) |
| A related employee becomes inactive after the profile is shown; the user selects their link | 404 `profile_not_available` (R27) | Application (C3 on every request) |
| Two users open the same profile | Independent reads; no contention | — |

### Existing data
This unit adds no rule to stored data. Records with a null department, designation or work phone are shown as "Not assigned" or "Not recorded" (A4). No migration or backfill is needed.

## 5. Contracts

| File | Type | Purpose | Machine-validated |
|---|---|---|---|
| `contracts/openapi.yaml` | OpenAPI 3.1 | `getEmployeeProfile`, schemas `EmployeeProfile`, `RelatedEmployee`, `ErrorEnvelope`, `ErrorBody` | Yes — Redocly CLI 1.25.11 `lint`: valid; 1 warning (no `info.license`) |

No SQL migration (Prisma owns the schema; this unit owns no entity). No AsyncAPI (no events).

## 6. UI

**Mockups:** none exist (confirmed by the user, 2026-11-24). This section is the specification; the frame inventory below is what a designer should produce from it.

### Screens

| Screen | Route | Purpose | API calls | R-IDs |
|---|---|---|---|---|
| Employee profile | `/employees/:employeeId` | Show one employee's directory information, manager and direct reports | `getEmployeeProfile` | R1–R27 |

**Layout (desktop ≥ 900 px; stacks to one column below 900 px and down to 320 px)**

```
[← Back to Employee List]

Arjun Mehta                                   (h1)
Accountant · Finance

┌ Details ──────────────────────────────────┐
│ Employee ID   E10234                      │
│ Designation   Accountant                  │
│ Department    Finance                     │
│ Work email    arjun.mehta@company.example │
│ Work phone    +44 20 7946 0123            │
└───────────────────────────────────────────┘
┌ Manager ─────────────┐ ┌ Direct reports (6) ──────────┐
│ Ravi Kumar (link)    │ │ Anil Shah (link)             │
│ Finance Manager      │ │ Accounts Assistant           │
└──────────────────────┘ │ Arjun Mehta (link) …         │
                         └──────────────────────────────┘
```

**Frame inventory (for the designer):** F1 populated, desktop · F2 populated, 360 px · F3 loading skeleton · F4 "No manager" and "No direct reports" · F5 missing values ("Not assigned", "Not recorded") · F6 manager as plain text (employee viewing own profile) · F7 "Profile not available" · F8 "Temporarily unavailable" with "Try again" · F9 offline.

### Components

All new components live in `src/features/employee-profile/` in the UI repo and are built on MUI 6.1 (Material UI).

| Component | Type | Inputs (type · required · default) | Events | States rendered |
|---|---|---|---|---|
| `EmployeeProfilePage` | page | route param `employeeId` (string · yes) | — | loading · populated · outcome |
| `ProfileDetails` | new | `profile: EmployeeProfile` (yes) | — | values, or "Not assigned" / "Not recorded" |
| `RelatedEmployeeItem` | new | `employee: RelatedEmployee` (yes) · `from: string \| undefined` (no · undefined) | — | link (`employeeId` not null) · plain text (null) |
| `ProfileOutcome` | new | `outcome: 'not_available' \| 'unavailable' \| 'offline'` (yes) · `onRetry: () => void` (required unless `not_available`) | `retry` | the three outcome panels |
| `BackToListLink` | new | `from: string \| undefined` (no · undefined) | — | link |
| `ProfileSkeleton` | new | — | — | loading placeholders |

Reused MUI components: `Link` (with React Router `Link` as `component`), `Typography`, `Card`, `List`, `ListItem`, `Skeleton`, `Alert`, `Button`.

### States — Employee profile

| State | What the user sees | R-IDs |
|---|---|---|
| loading | The "Back to Employee List" link and `ProfileSkeleton` (heading, details and two sections as skeleton blocks); the main region has `aria-busy="true"` | R16 |
| empty — first use | N/A — because a profile always has a subject; otherwise it is the "not available" state | — |
| empty — filtered | N/A — because the profile has no search or filter | — |
| empty — sections | "No manager" (R6, R7) and "No direct reports" (R8) within a populated profile | R6, R7, R8 |
| populated | Heading, details, manager, direct reports (A–Z, all shown); related employees as links or plain text | R2–R5, R13, R14, R20 |
| partial | N/A — because the API is all-or-nothing (A3); a failure shows the error state, never part of a profile | R24 |
| submitting | N/A — because the profile is view only | — |
| success | N/A — because there is no mutation | — |
| error — recoverable | 503 / 500 / network: "Profile temporarily unavailable" with "Try again" and "Back to Employee List" | R24 |
| error — terminal | 404: "Profile not available" with "Back to Employee List"; no other detail | R17–R19, R21, R22, R25–R27 |
| disabled / unauthorized | Same as error — terminal (the uniform refusal, Q4). Related employees outside the permitted set are plain text, not disabled links | R14, R17–R19 |
| offline — on load | "You're offline. Check your connection and try again." with "Try again"; retries automatically when back online | R24 |
| offline — on submit | N/A — because there is no submit | — |
| session expiry mid-flow | 401 on load: **UNRESOLVED** (Q-D2) — sign-in flow not designed. There is no user work to preserve (view only) | — |
| stale view | The profile is not refreshed while open: no polling and no refetch on focus; a reload or a new open reads again | R23 |

### Validation & copy

No input validation: the profile has no form fields.
Copy keys live in `src/locales/en/profile.json` (de and fr translations UNRESOLVED, Q-D4).

| Key | English |
|---|---|
| `profile.back` | Back to Employee List |
| `profile.section.details` | Details |
| `profile.section.manager` | Manager |
| `profile.section.directReports` | Direct reports ({{count}}) |
| `profile.field.employeeId` | Employee ID |
| `profile.field.designation` | Designation |
| `profile.field.department` | Department |
| `profile.field.workEmail` | Work email |
| `profile.field.workPhone` | Work phone |
| `profile.value.notAssigned` | Not assigned |
| `profile.value.notRecorded` | Not recorded |
| `profile.noManager` | No manager |
| `profile.noDirectReports` | No direct reports |
| `profile.loading` | Loading profile |
| `profile.notAvailable.title` | Profile not available |
| `profile.notAvailable.body` | This profile can't be shown. |
| `profile.unavailable.title` | Profile temporarily unavailable |
| `profile.unavailable.body` | We couldn't load this profile. Try again in a moment. |
| `profile.offline.title` | You're offline |
| `profile.offline.body` | Check your connection and try again. |
| `profile.retry` | Try again |

No dates are shown.

### API error → UI

| Received | User sees | Focus goes to |
|---|---|---|
| 404 `profile_not_available` | `ProfileOutcome outcome="not_available"` | The outcome heading "Profile not available" |
| 503 `employee_details_unavailable` | `ProfileOutcome outcome="unavailable"` with "Try again" | The outcome heading |
| 500 `internal_error` | Same as 503 | The outcome heading |
| Network failure, `navigator.onLine === false` | `ProfileOutcome outcome="offline"` | The outcome heading |
| 401 `unauthenticated` | UNRESOLVED (Q-D2) | — |

### Accessibility (WCAG 2.2 AA)
- **Keyboard:** Tab order is: Back link → manager link (if any) → direct-report links in list order → outcome "Try again" (if shown). Enter activates links and buttons. No composite widgets, no Esc behaviour.
- **Focus on route entry:** the `<h1>` (employee name, `tabIndex=-1`) once loaded; during loading, focus stays on the document and the main region is `aria-busy`.
- **After an error, and after each terminal outcome:** the outcome heading (`tabIndex=-1`), which announces the outcome.
- **After "Try again" succeeds:** the `<h1>`. After it fails again: the outcome heading.
- **Semantics:** details as a description list (`<dl>`); direct reports as a list (`<ul>` with a count in the section heading); manager and direct-report links have accessible names "{name}, {designation}"; plain-text related employees are not focusable.
- **Visual:** MUI default theme contrast ≥ 4.5:1 for text and ≥ 3:1 for UI; link targets ≥ 24×24 CSS px; usable at 200% zoom; no horizontal scroll at 320 px; the skeleton animation is disabled under `prefers-reduced-motion`; meaning never carried by colour alone.
- **Knowingly not covered:** de and fr copy (Q-D4); the 1.4× German text expansion is checked against layout only once translations exist.

### Test IDs

| `data-testid` | Element |
|---|---|
| `profile-page` | Page root |
| `profile-back-link` | Back to Employee List |
| `profile-loading` | Skeleton container |
| `profile-name` | `<h1>` |
| `profile-employee-id` | Employee ID value |
| `profile-designation` | Designation value |
| `profile-department` | Department value |
| `profile-work-email` | Work email value |
| `profile-work-phone` | Work phone value |
| `profile-manager` | Manager section |
| `profile-manager-link` | Manager as a link |
| `profile-manager-text` | Manager as plain text |
| `profile-no-manager` | "No manager" |
| `profile-direct-reports` | Direct reports list |
| `profile-direct-report-item` | Each direct report (repeated) |
| `profile-direct-report-link` | Link inside an item |
| `profile-direct-report-text` | Plain text inside an item |
| `profile-no-direct-reports` | "No direct reports" |
| `profile-not-available` | Terminal outcome panel |
| `profile-unavailable` | Recoverable outcome panel |
| `profile-offline` | Offline outcome panel |
| `profile-retry-button` | Try again |

### Must NOT offer

| Affordance | Why |
|---|---|
| Any edit control | View only; editing is U002 (OUT-6) |
| Salary, bank details or personal information | OUT-1 |
| The manager's manager, or indirect reports | OUT-2 |
| Inactive employees, as subject or related employee | OUT-3, IN-5 |
| "Show more" or paging on direct reports | Q5 |
| Auto-refresh or live updates | R23 |
| Different messages for "not found" and "not permitted" | Q4 |
| Search or filters on the profile page | OUT-5 |
| A link on a related employee outside the viewer's permitted set | Q2, R14 |
| Email or call actions (mailto, tel) | Not asked for |

## 7. Implementation

**All repos are new (greenfield), confirmed by the user on 2026-11-24. The stack below is Decision D1, confirmed by Shiny Priscilla J on 2026-11-24.**

### 7.1 API repo — `employee-management-api` (proposed local path `C:\acxworks\employee-management-api`)

**Stack:** Node.js 22 LTS · TypeScript 5.6 · NestJS 10.4 · Prisma ORM 5.22 with PostgreSQL 16 · Jest 29 (unit) · Supertest 7 (HTTP) · jest-openapi 0.14 (response-to-contract checks) · Testcontainers 10 (`@testcontainers/postgresql`) for acceptance tests against a real PostgreSQL.

**Layout**
```
prisma/schema.prisma                 Employee model (owner UNRESOLVED, Q-D1)
src/main.ts                          bootstrap; global ApiErrorFilter; /api/v1 prefix
src/app.module.ts
src/prisma/prisma.service.ts         shared PrismaClient
src/common/auth/                     AuthGuard, @CurrentViewer(), Viewer type (mechanism UNRESOLVED)
src/common/errors/                   ApiErrorFilter, DomainError
src/employees/employees.module.ts
src/employees/profile/               this unit
test/contract/                       contract tests
test/acceptance/                     acceptance tests per R-ID
```

**Implementation map**

| Design element | Class / function | File | New / modified | R-IDs |
|---|---|---|---|---|
| C1 Profile endpoint · `getEmployeeProfile` | `EmployeeProfileController.getEmployeeProfile()` | `src/employees/profile/employee-profile.controller.ts` | new | R1, R16–R27 |
| C2 Profile assembler | `EmployeeProfileService.getProfile()` | `src/employees/profile/employee-profile.service.ts` | new | R2–R8, R13, R14, R20 |
| C3 Permitted-set policy | `PermittedSetPolicy.canOpen()` | `src/employees/profile/permitted-set.policy.ts` | new | R9–R14, R17–R19, R21, R22, R25–R27 |
| C4 Profile snapshot reader | `EmployeeProfileRepository.loadSnapshot()` | `src/employees/profile/employee-profile.repository.ts` | new | R2–R5, R16, R23, R24 |
| Response shapes | `EmployeeProfileDto`, `RelatedEmployeeDto` | `src/employees/profile/employee-profile.dto.ts` | new | R2–R4, R13, R14 |
| Reason codes | `ProfileNotAvailableError`, `EmployeeDetailsUnavailableError` | `src/employees/profile/employee-profile.errors.ts` | new | R17–R27 |
| C5 Viewer resolver | `AuthGuard`, `CurrentViewer`, `Viewer` | `src/common/auth/auth.guard.ts`, `src/common/auth/current-viewer.decorator.ts`, `src/common/auth/viewer.ts` | new | R9–R12, R19 |
| C6 Error envelope | `ApiErrorFilter`, `DomainError` | `src/common/errors/api-error.filter.ts`, `src/common/errors/domain-error.ts` | new | R17–R27 |
| Module wiring | `EmployeesModule` | `src/employees/employees.module.ts` | new | — |
| `Employee` entity (read) | Prisma `model Employee` + index `(managerEmployeeId, status)` | `prisma/schema.prisma` | new — owner UNRESOLVED (Q-D1) | R2–R5, R16 |

**Tests**

| Kind | Location / naming | Covers |
|---|---|---|
| Unit | `*.spec.ts` beside the source (e.g. `permitted-set.policy.spec.ts`) | C2, C3 rules: every line of `canOpen`, sorting, null manager, blanking `employeeId` |
| Contract | `test/contract/employee-profile.contract-spec.ts` | Every response validated against `contracts/openapi.yaml` (jest-openapi), including 404 and 503 examples |
| Acceptance | `test/acceptance/employee-profile.e2e-spec.ts`, one `it('R<n> …')` per R-ID | R2–R14, R16–R27 against seeded PostgreSQL (Testcontainers) |
| Performance | `test/performance/employee-profile.perf-spec.ts` | R16 server budget: p95 ≤ 300 ms with 5,000 employees and 50 reports |

### 7.2 UI repo — `employee-management-web` (proposed local path `C:\acxworks\employee-management-web`)

**Stack:** TypeScript 5.6 · React 18.3 · Vite 5.4 · React Router 6.26 · TanStack Query 5.59 · MUI 6.1 (design system) · i18next 23 + react-i18next 15 · openapi-typescript 7.4 (types generated from `contracts/openapi.yaml`) · Vitest 2.1 + React Testing Library 16 + MSW 2.4 (unit) · Playwright 1.48 (end-to-end) · @axe-core/playwright 4.10 (accessibility checks).

**Layout**
```
src/app/router.tsx                       routes
src/api/schema.d.ts                      generated from contracts/openapi.yaml
src/api/client.ts                        apiFetch, ApiError
src/features/employee-profile/           this unit
src/features/employee-list/              U003 (not yet designed)
src/locales/en/profile.json              copy keys
e2e/                                     Playwright specs
```

**Implementation map**

| Design element | Class / function / component | File | New / modified | R-IDs |
|---|---|---|---|---|
| Route `/employees/:employeeId` | `router` entry | `src/app/router.tsx` | new | R1 |
| API client | `apiFetch()`, `ApiError` | `src/api/client.ts` | new | R17–R27 |
| Generated types | `paths`, `components` | `src/api/schema.d.ts` | new (generated) | — |
| Profile query | `getEmployeeProfile()`, `useEmployeeProfile()` (TanStack Query; `staleTime: 0`, `refetchOnWindowFocus: false`, `retry: false`) | `src/features/employee-profile/api/getEmployeeProfile.ts`, `src/features/employee-profile/hooks/useEmployeeProfile.ts` | new | R16, R23, R24 |
| C7 Employee profile screen | `EmployeeProfilePage` | `src/features/employee-profile/EmployeeProfilePage.tsx` | new | R1–R27 |
| Details | `ProfileDetails` | `src/features/employee-profile/components/ProfileDetails.tsx` | new | R2, R20 |
| Related employee | `RelatedEmployeeItem` | `src/features/employee-profile/components/RelatedEmployeeItem.tsx` | new | R3, R4, R13, R14 |
| Outcomes | `ProfileOutcome` | `src/features/employee-profile/components/ProfileOutcome.tsx` | new | R17–R19, R21, R22, R24–R27 |
| C8 Back link | `BackToListLink` | `src/features/employee-profile/components/BackToListLink.tsx` | new | R15 |
| Loading | `ProfileSkeleton` | `src/features/employee-profile/components/ProfileSkeleton.tsx` | new | R16 |
| Copy | `profile.*` keys | `src/locales/en/profile.json` | new | R6–R8, R20 |
| C9 List-row link | U003 list row component | `src/features/employee-list/…` (U003) | modified — dependency (Q-D3) | R1, R15 |

**Tests**

| Kind | Location / naming | Covers |
|---|---|---|
| Unit / component | `*.test.tsx` beside the component; MSW handlers in `src/test/msw/handlers.ts` | Each state row, copy, link vs plain text, focus after error |
| End-to-end | `e2e/employee-profile.spec.ts`, one `test('R<n> …')` per user-visible R-ID; API mocked via Playwright routes from the OpenAPI examples | R1–R15, R17–R27; selects by `data-testid` only |
| Accessibility | `e2e/employee-profile.a11y.spec.ts` (axe) | Populated, not-available and unavailable states |

## 8. Cross-cutting

| Concern | This unit |
|---|---|
| Tenant isolation | None — because a single organisation is assumed. Project baseline item open (owner Shiny Priscilla J) |
| Authentication | **UNRESOLVED** (Q-D2): left out by the user on 2026-11-24. The design depends only on `Viewer { employeeId, isHr }`. **The unit must not go live with a stub `AuthGuard`** (risk K1) |
| Authorization | Permitted-set policy C3 on every request, for the subject and each related employee; uniform 404 (Q4) |
| Validation | Path ID: length 1–64, otherwise 404 with no database read. No other input |
| Error model | Shared envelope (proposed); codes `profile_not_available`, `employee_details_unavailable`, plus baseline `unauthenticated` and `internal_error` |
| Data exposure | Directory fields only; `employeeId` of a related employee withheld unless openable; `no-store` caching; no names, emails or phones in logs |
| Observability | Metrics: `employee_profile_requests_total{outcome=ok\|not_available\|unavailable\|error}`, `employee_profile_duration_seconds` histogram. Log fields: `requestId`, `viewerEmployeeId`, `subjectEmployeeId`, `outcome`, `durationMs`, `directReportCount`. Spans: HTTP request → `loadSnapshot` (database transaction). A "profile won't open" complaint can be diagnosed from `outcome` plus the IDs, without a database query |
| Performance | R16: 2 s at p95 end-to-end (requirements §5 note). Server budget p95 ≤ 300 ms; database statement timeout 1,500 ms → 503. **p99: UNRESOLVED** (baseline) |
| Availability | Project baseline item open — UNRESOLVED |
| Rate limits | Project baseline item open — UNRESOLVED |
| Accessibility | WCAG 2.2 AA (§6) |
| Migration / backfill | None — because this unit adds no rule to stored data |
| Feature flag | None — because this is a new read-only screen with nothing in flight. Until sign-in exists, the route is simply not deployed |
| Failure recovery | Stateless reads; recovery is a retry. Nothing to reconcile |

## 9. Decisions & risks

### Decisions

| # | Decision | Alternative rejected | Why | Status |
|---|---|---|---|---|
| D1 | Stack in §7: Node.js 22 / NestJS 10 / Prisma 5 / PostgreSQL 16 for the API; React 18 / Vite 5 / MUI 6 / TanStack Query 5 for the UI; one language (TypeScript) across both | Python (FastAPI) or Java (Spring Boot) for the API | One language for both repos; types generated from the same OpenAPI; mainstream and well supported | **Confirmed** — Shiny Priscilla J, 2026-11-24 |
| D2 | One server-built profile endpoint | UI composes several calls | §1: no data leak, all-or-nothing, one round trip | Proposed |
| D3 | Uniform 404 `profile_not_available` | 403 for "not permitted" | Q4: existence must not be revealed | Proposed (from requirements) |
| D4 | Related employee `employeeId` is null when not openable | Send the ID plus a `canOpen` flag | Doesn't hand out IDs the viewer can't use; one field instead of two that can disagree | Proposed |
| D5 | Prisma owns the schema; no SQL contract | Hand-written SQL migrations | Greenfield; one schema owner; migrations generated | Proposed |
| D6 | `directReports` embedded and unpaged | Cursor-paged list (contracts standard) | Q5 requires all shown; ~50 rows (A1) | Proposed — deviation from standard |
| D7 | The list's search, filters and page live in the list URL query string; the profile carries it as router state `from` | Global client-side store | Survives reload and new tabs; no shared state | Proposed — **U003 design must adopt it** (Q-D3) |
| D8 | Accept that response timing may differ between unknown and not-permitted IDs | Constant-time padding | The response body is identical; directory data is low sensitivity; padding costs R16 latency | Proposed — risk accepted |

### Open items

| # | Item | Type | Blocks | Owner |
|---|---|---|---|---|
| RQ1 | Intent IN-8 and OUT-4 say "consume an existing `GET /employees/{id}`"; the API repo is new, so this unit builds it. Amend OUT-4 | Requirements issue | Nothing (design proceeds) | `ba-unit-requirements` |
| Q-D1 | Who defines the shared `Employee` Prisma model and its `(managerEmployeeId, status)` index — U002, U003 or a foundation unit? | UNRESOLVED | C4, `prisma/schema.prisma` | Shiny Priscilla J |
| Q-D2 | Sign-in mechanism, how `Viewer { employeeId, isHr }` is obtained, 401 / session-expiry UX | UNRESOLVED (deferred by the user) | C5; go-live | Shiny Priscilla J |
| Q-D3 | U003 list rows must link to `/employees/{id}`, pass `from`, and restore state from the URL | Dependency | R1, R15 | Architect (U003 design) |
| Q-D4 | de and fr translations of the copy keys | UNRESOLVED | Locales only | Shiny Priscilla J |
| Q-D5 | API host; repo names and local paths (proposed in §7) | UNRESOLVED | Deployment only | Shiny Priscilla J |
| Q6 / A1 | Largest number of direct reports; assumed 50 | Carried assumption | R4, R16 sizing | Shiny Priscilla J |
| Q7 | Any other role or service using the profile; assumed none | Carried question | C3 | Shiny Priscilla J |
| A2 | No employee record and no HR role, or an inactive own record, means no permitted set | Carried assumption | C3 rules 2 and 4 | Shiny Priscilla J |
| A3 | No partial profile | Carried assumption | C2, C4, UI "partial" | Shiny Priscilla J |
| A4 | "Not assigned" / "Not recorded" | Carried assumption | `ProfileDetails` | Shiny Priscilla J |
| Baseline | Availability, p99, rate limits, tenant isolation, data classification, retention, session-expiry UX | Open baseline items | §8 rows | Shiny Priscilla J |

### Risks (costliest to find after launch first)

| # | Risk | Response |
|---|---|---|
| K1 | Shipping with a stub `AuthGuard` exposes every profile | Mitigate: the guard throws unless configured; the route is not deployed until Q-D2 is resolved |
| K2 | The `Employee` model is built differently by U002/U003 (for example a role field instead of `designation`) | Mitigate: Q-D1 before tasks; the field table in §4 is the contract |
| K3 | U003 keeps list state outside the URL, breaking R15 | Mitigate: Q-D3 handed to U003 design |
| K4 | Timing side channel on 404 | Accept (D8) |

## 10. Requirement coverage

| R-ID | Design | API | UI | Implementation |
|---|---|---|---|---|
| R1 | F1 steps 1–2; C9 | `getEmployeeProfile` 200 | Route; list-row link | `router.tsx`; U003 row (Q-D3) |
| R2 | C2; §3 response | 200 fields | `ProfileDetails` | `employee-profile.service.ts`; `ProfileDetails.tsx` |
| R3 | C2 manager | `manager` | Manager section | service; `RelatedEmployeeItem.tsx` |
| R4 | C2 sort; F1 step 6 | `directReports` | Direct reports list | service; `RelatedEmployeeItem.tsx` |
| R5 | C4 filter on ACTIVE | `directReports` | Direct reports list | `employee-profile.repository.ts` |
| R6 | C2 null for inactive manager | `manager: null` | "No manager" | service; `EmployeeProfilePage.tsx` |
| R7 | C2 null when no manager | `manager: null` | "No manager" | service; page |
| R8 | — | `directReports: []` | "No direct reports" | page |
| R9 | C3 rule 3 | 200 | populated | `permitted-set.policy.ts` |
| R10 | C3 rules 5–6 | 200 | populated | policy |
| R11 | C3 rule 5 | 200 | populated | policy |
| R12 | C3 rule 3 | 200 | populated | policy |
| R13 | C2 + C3 on related; F3 | `RelatedEmployee.employeeId` set | link | service; `RelatedEmployeeItem.tsx` |
| R14 | C2 + C3 on related | `RelatedEmployee.employeeId: null` | plain text | service; `RelatedEmployeeItem.tsx` |
| R15 | F4; D7 | — | `BackToListLink` | `BackToListLink.tsx`; U003 (Q-D3) |
| R16 | C4 index and timeout; §8 performance | 200 | loading | repository; `perf-spec` |
| R17 | C3 rule 7; F2 | 404 `profile_not_available` | not available | policy; controller |
| R18 | C3 rule 7; F2 | 404 | not available | policy |
| R19 | C3 rule 4; F2 | 404 | not available | policy |
| R20 | C2 nulls; A4 | nullable fields | "Not assigned" / "Not recorded" | `ProfileDetails.tsx` |
| R21 | C3 rule 1; F2 | 404 | not available | policy |
| R22 | F2 step 1 (length check) and rule 1 | 404 | not available | controller; policy |
| R23 | Concurrency matrix; no refetch | — | stale view | `useEmployeeProfile.ts` |
| R24 | C4 timeout; F1 step 4 failure | 503 `employee_details_unavailable` | unavailable + Try again | repository; `ProfileOutcome.tsx` |
| R25 | C3 rule 1 | 404 | not available | policy |
| R26 | C3 rule 2 | 404 | not available | policy |
| R27 | F3 failure; C3 rule 1 | 404 | not available | policy |

**Field rules → where enforced:**
- Profile subject → C3 and F2.
- Directory information → C2 and `ProfileDetails`.
- Immediate manager → C2.
- Direct reports → C2 and C4.
- Related-employee link → C2/C3 and `RelatedEmployeeItem`.
- "Who may change it" → nothing is writable (§3).

**Reason codes:**

| Code | API error row | UI row |
|---|---|---|
| `profile_not_available` | 404 | not available |
| `employee_details_unavailable` | 503 | unavailable |

Every component, endpoint and screen has an implementation-map row.

## 11. Change log

| Date | Version | By | Change |
|---|---|---|---|
| 2026-11-24 | v1 | @architect | First version; approved by Shiny Priscilla J, stack D1 confirmed |
