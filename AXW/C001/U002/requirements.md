# Requirements — Update Employee Information

| | |
|---|---|
| **Project** | AXW — axworks3 |
| **Capability** | AXW-C001 — Features |
| **Unit** | AXW-C001-U002 |
| **Built from** | intent.md, ref 2b9b1003411ca62fd678d1c01e8cf38d4c4920a1 |
| **Approved by** | ArunVijay |
| **Approved on** | 2026-11-20 |

> What the system must do, observable at its boundary. No endpoints, tables, frameworks or status codes.
> R-IDs are permanent: never renumber, reuse or delete.

## 1. Functional requirements

| R-ID | Requirement (one testable statement) | Traces to | Priority | Readiness |
|---|---|---|---|---|
| R1 | An active employee changes their own mobile number, and the new value takes effect when saved, with no approval. | IN-1, O1, O2 | must | Ready with assumptions (A1) |
| R2 | An active employee changes their own personal email, and the new value takes effect when saved, with no approval. | IN-1, O1, O2 | must | Ready |
| R3 | An active employee submits a change to their own residential address; it is recorded as a pending change and the current address is unchanged until an HR user approves it. | IN-2, O1, O2, O5 | must | Ready |
| R4 | An active employee submits a change to their own emergency contact; it is recorded as a pending change and the current emergency contact is unchanged until an HR user approves it. | IN-2, O1, O2, O5 | must | Ready |
| R5 | An employee sees each change they submitted to their own record with its state (pending, approved, rejected or withdrawn) and, for a rejected change, the rejection reason. | IN-2, IN-4, O2 | must | Ready |
| R6 | An active employee withdraws their own pending change; it is never applied and can no longer be decided. | IN-2, O5 | must | Ready |
| R7 | An HR user submits a change to an employee's department, designation, reporting manager, salary or bank details; it is recorded as a pending change that only a different HR user can decide. | IN-3, O5 | must | Ready with assumptions (A6) |
| R8 | An HR user submits a change to an employee's residential address or emergency contact; it is recorded as a pending change that only a different HR user can decide. | IN-3, O5 | must | Ready |
| R9 | An HR user changes an employee's mobile number or personal email, and the new value takes effect when saved, with no approval. | IN-3, O1 | must | Ready |
| R10 | An HR user sees every pending change awaiting a decision, each showing the employee, the field, the current value, the proposed value, who submitted it and when. | IN-4, O2, O5 | must | Ready |
| R11 | An HR user approves a pending change; the proposed value takes effect immediately and the change becomes approved. | IN-4, O2, O5 | must | Ready |
| R12 | An HR user rejects a pending change with a reason; the value is not changed, the change becomes rejected and the reason is visible to the employee (R5). | IN-4, O5 | must | Ready |
| R13 | When the field's current value differs from the value at the time the change was submitted, the approving HR user is shown the difference and must confirm before the change is applied. | IN-4, O5 | should | Ready |
| R14 | A mobile number is accepted only as a valid international number that includes a country code. | IN-5, O3 | must | Ready with assumptions (A1) |
| R15 | A personal email is accepted only if it is a well-formed email address and is not the employee's work email. | IN-5, O3 | must | Ready |
| R16 | A residential address is accepted only if it has address line 1, city, postal code and country. | IN-5, O3 | must | Ready with assumptions (A1) |
| R17 | An emergency contact is accepted only as one contact with a name, a relationship and a valid phone number. | IN-5, O3 | must | Ready with assumptions (A1) |
| R18 | Bank details are accepted only with an account holder name, an account number and a bank or branch code, each in a valid format; the account is not verified with a bank. | IN-5, O3 | must | Ready with assumptions (A2) |
| R19 | A reporting manager is accepted only if they are an active employee who is neither the employee nor anyone in the employee's reporting line below them. | IN-5, O3 | must | Ready with assumptions (A5) |
| R20 | A salary is accepted only as a positive amount in the organisation's currency. | IN-5, O3 | must | Ready with assumptions (A3) |
| R21 | A department or designation is accepted only if it is one of the organisation's existing departments or designations. | IN-5, O3 | must | Ready with assumptions (A4) |
| R22 | Every applied change, submission, approval, rejection and withdrawal is added to the change record with the employee, the field, the old value, the new value, who acted, in what role, and when. | IN-6, O4 | must | Ready |
| R23 | An HR user sees the full change record for any employee. | IN-6, O4 | must | Ready |
| R24 | An employee sees the change record for their own record, with salary and bank-detail values masked. | IN-6, O4 | should | Ready |
| R25 | A change by an employee to any record other than their own is refused with `not_own_record`; this applies equally to a manager acting on a reporting employee. | IN-7, O5 | must | Ready |
| R26 | A change by an employee to department, designation, reporting manager, salary or bank details is refused with `field_not_permitted`. | IN-7, O5 | must | Ready |
| R27 | Viewing pending changes awaiting a decision, approving or rejecting by anyone who is not an HR user is refused with `not_hr_user`. | IN-7, IN-4, O5 | must | Ready |
| R28 | An HR user's decision on a pending change they submitted, or on a change to their own record, is refused with `self_approval_not_allowed`. | IN-7, O5 | must | Ready with assumptions (A6) |
| R29 | Any change or withdrawal by an employee whose own record is inactive is refused with `employee_inactive`. | IN-8, O5 | must | Ready |
| R30 | An HR user changes an inactive employee's record under the same rules as an active employee's (R7–R9, R11–R13). | IN-8, O5 | must | Ready |
| R31 | A new change to a field that already has a pending change is refused with `change_already_pending`, whoever submitted the first. | IN-2, IN-3, O5 | must | Ready with assumptions (A7) |
| R32 | A save based on a version of the employee record that has since changed is refused with `record_changed_since_viewed`; nothing is changed. | IN-5, O3 | must | Ready |
| R33 | Approving, rejecting or withdrawing a change that is no longer pending is refused with `change_already_decided`; nothing is changed. | IN-4, O5 | must | Ready |
| R34 | A change takes effect only together with its change-record entry; if either cannot be recorded, neither is, and the user is told `change_not_saved`. | IN-6, O4 | must | Ready |
| R35 | A save whose value is identical to the current value, or a repeat of the same request, changes nothing, adds no change-record entry and is reported as successful. | IN-6, O4 | must | Ready with assumptions (A20) |
| R36 | While a save, submission or decision is in progress, the user cannot send it again. | IN-1, IN-2, O3 | must | Ready |
| R37 | If the connection is lost or the session expires before a submission is confirmed, the values the user typed are kept and the user is told the change was not confirmed. | IN-1, IN-2, O2 | should | Ready with assumptions (A17) |
| R38 | A user leaving with unsaved changes is warned and can stay or discard them. | IN-1, IN-2, O3 | should | Ready |
| R39 | A user is offered for editing only the records and fields they are permitted to change. | IN-7, O5 | must | Ready |

Priority: `must` · `should` · `could`.
Readiness: `Ready` · `Ready with assumptions (A<n>)` · `Not ready — Q<n>`.

## 2. Acceptance criteria

### R1 · R2 · R9 — immediate changes
- **Given** an active employee **when** they save a valid new mobile number for their own record **then** the new value is in effect and one change-record entry exists (R22).
- **Given** an HR user **when** they save a valid new personal email for any employee **then** the new value is in effect with no pending change created.

| Failure class | Case | Expected result |
|---|---|---|
| input | value missing or malformed | Refused with `invalid_mobile_number` / `invalid_personal_email`; nothing changed (R14, R15) |
| input | personal email equals work email | Refused with `personal_email_same_as_work_email`; nothing changed |
| input | employee does not exist | Refused with `employee_not_found`; nothing changed |
| authorization | not signed in | Refused with `not_signed_in` |
| authorization | employee or manager targets another record | Refused with `not_own_record` (R25) |
| repetition | double submit or client retry | One change, one change-record entry (R35) |
| concurrency | record changed since the user opened it | Refused with `record_changed_since_viewed` (R32) |
| partial failure | change saved but change record cannot be written | Neither kept; `change_not_saved` (R34) |
| state | employee's own record is inactive (R1, R2) | Refused with `employee_inactive` (R29) |
| state | HR edits an inactive employee (R9) | Allowed (R30) |

**Not applicable:** dependency (all data held in this system, A13; sign-in failure is covered by `not_signed_in`), ordering (no events consumed), time (no date windows; change takes effect at save, A18).

### R3 · R4 · R8 — submitting a pending change
- **Given** an active employee with no pending address change **when** they submit a valid new address **then** a pending change exists, the current address is unchanged, and the employee sees it as pending (R5).

| Failure class | Case | Expected result |
|---|---|---|
| input | address or emergency contact incomplete or invalid | Refused with `address_incomplete` / `emergency_contact_incomplete` / `invalid_emergency_contact_phone`; no pending change (R16, R17) |
| input | employee does not exist | Refused with `employee_not_found` |
| authorization | not signed in | Refused with `not_signed_in` |
| authorization | employee targets another record | Refused with `not_own_record` |
| repetition | same submission repeated | One pending change (R35) |
| repetition | a different change for a field already pending | Refused with `change_already_pending` (R31) |
| concurrency | two submissions for the same field at once | One becomes pending; the other is refused with `change_already_pending` |
| partial failure | pending change recorded without its change-record entry | Neither kept; `change_not_saved` (R34) |
| state | employee's own record is inactive | Refused with `employee_inactive` (R29) |

**Not applicable:** dependency (A13), ordering (no events), time (pending changes do not expire, A8).

### R6 — withdrawing
- **Given** an employee's own pending change **when** they withdraw it **then** it becomes withdrawn, the value is unchanged and it no longer appears to HR for a decision.

| Failure class | Case | Expected result |
|---|---|---|
| input | change does not exist | Refused with `change_not_found` |
| authorization | change belongs to another employee | Refused with `not_own_record` |
| repetition | withdraw sent twice | Withdrawn once; the repeat is reported as successful (R35) |
| concurrency | withdrawal at the same time as an HR decision | Whichever lands first wins; the other is refused with `change_already_decided` |
| state | change already approved, rejected or withdrawn | Refused with `change_already_decided` (R33) |
| state | employee became inactive while the change was pending | Refused with `employee_inactive`; the change stays pending for HR (D12) |

**Not applicable:** dependency (A13), partial failure (single state change, covered by R34), ordering (no events), time (no expiry, A8).

### R7 — HR submits a change to an HR-only field
- **Given** HR user A **when** they submit a valid new salary for an employee **then** a pending change exists that HR user A cannot decide and any other HR user can.

| Failure class | Case | Expected result |
|---|---|---|
| input | invalid department, designation, reporting manager, salary or bank details | Refused with that field's code (R18–R21); no pending change |
| input | employee does not exist | Refused with `employee_not_found` |
| authorization | non-HR user | Refused with `field_not_permitted` (R26) |
| repetition | field already has a pending change | Refused with `change_already_pending` (R31) |
| concurrency | two HR users submit for the same field | One pending; the other is refused with `change_already_pending` |
| partial failure | change-record write fails | `change_not_saved` (R34) |
| state | employee is inactive | Allowed (R30) |

**Not applicable:** dependency (A13), ordering (no events), time (no effective dating, A18).

### R11 — approve
- **Given** a pending change submitted by someone else, not concerning the approver's own record **when** an HR user approves it **then** the proposed value is in effect, the change is approved, and the employee sees it as approved.

| Failure class | Case | Expected result |
|---|---|---|
| input | proposed value no longer valid (e.g. new reporting manager now inactive) | Refused with that field's validation code; the change stays pending and can be rejected |
| input | change does not exist | Refused with `change_not_found` |
| authorization | not an HR user | Refused with `not_hr_user` (R27) |
| authorization | approver submitted it or is the employee concerned | Refused with `self_approval_not_allowed` (R28) |
| repetition | approve sent twice | Applied once (R35) |
| concurrency | two HR users decide at once, or approval races a withdrawal | First wins; the other gets `change_already_decided` |
| partial failure | value applied but the decision or change record not recorded | None of the three kept; `change_not_saved` (R34) |
| state | already decided or withdrawn | Refused with `change_already_decided` |
| state | employee became inactive while pending | Approval allowed (D12) |
| state | current value changed since submission | Confirmation required first: `approval_confirmation_required` (R13) |

**Not applicable:** dependency (A13), ordering (no events), time (no expiry, A8).

### R12 — reject
- **Given** a pending change **when** an HR user rejects it with a reason **then** the value is unchanged, the change is rejected and the employee sees the reason.

| Failure class | Case | Expected result |
|---|---|---|
| input | reason empty | Refused with `rejection_reason_required` |
| input | change does not exist | Refused with `change_not_found` |
| authorization | not an HR user / self-decision | `not_hr_user` / `self_approval_not_allowed` |
| repetition | reject sent twice | Rejected once (R35) |
| concurrency | races another decision or a withdrawal | First wins; the other gets `change_already_decided` |
| partial failure | change-record write fails | `change_not_saved` (R34) |
| state | already decided or withdrawn | Refused with `change_already_decided` |

**Not applicable:** dependency (A13), ordering (no events), time (no expiry, A8).

### R14–R21 — validation
Each rule is applied when the change is submitted and again when it is approved (R11).

| Rule | Refused with |
|---|---|
| R14 mobile not a valid international number with country code | `invalid_mobile_number` |
| R15 personal email malformed | `invalid_personal_email` |
| R15 personal email equals work email | `personal_email_same_as_work_email` |
| R16 address missing line 1, city, postal code or country | `address_incomplete` |
| R17 emergency contact missing name, relationship or phone | `emergency_contact_incomplete` |
| R17 emergency contact phone invalid | `invalid_emergency_contact_phone` |
| R18 bank details missing a part | `bank_details_incomplete` |
| R18 account number or bank/branch code in the wrong format | `invalid_bank_details_format` |
| R19 manager not active | `reporting_manager_not_active` |
| R19 manager is the employee or someone below them | `reporting_manager_creates_cycle` |
| R20 salary not a positive amount | `invalid_salary_amount` |
| R21 unknown department | `unknown_department` |
| R21 unknown designation | `unknown_designation` |

**Not applicable:** authorization, repetition, concurrency, partial failure, state (covered by the requirement each rule is applied in), dependency (lists held in this system, A4/A13), ordering (no events), time (no date fields validated).

### R22 — change record
- **Given** any change, submission, approval, rejection or withdrawal **when** it is completed **then** exactly one change-record entry exists for it, with the employee, field, old value, new value, actor, actor's role and time.

| Failure class | Case | Expected result |
|---|---|---|
| repetition | repeated request | No second entry (R35) |
| partial failure | entry cannot be written | The action is not kept; `change_not_saved` (R34) |
| time | when the action happened | Recorded in one reference time zone and shown in the viewer's local time (A21) |

**Not applicable:** input (no user input of its own), authorization (R23, R24), concurrency (one entry per completed action), dependency (A13), ordering (no events), state (entries are never changed or deleted).

### R24 — employee's view of their own history
- **Given** an employee **when** they view their own change record **then** salary and bank-detail values show as masked, and all other fields show in full.

| Failure class | Case | Expected result |
|---|---|---|
| authorization | employee asks for another employee's change record | Refused with `not_own_record` |
| authorization | not signed in | Refused with `not_signed_in` |

**Not applicable:** input (no input beyond own record), repetition (read only), concurrency (read only), dependency (A13), partial failure (read only), ordering (no events), time (no windows), state (an inactive employee has no access, R29).

### Reason codes

| Code | Meaning (business terms) | R-ID |
|---|---|---|
| `not_signed_in` | The user is not signed in. | R1–R12, R24 |
| `not_own_record` | An employee or manager acted on a record that is not their own. | R25 |
| `field_not_permitted` | The user may not change this field. | R26 |
| `not_hr_user` | Only an HR user may view or decide pending changes awaiting a decision. | R27 |
| `self_approval_not_allowed` | An HR user tried to decide a change they submitted, or one to their own record. | R28 |
| `employee_inactive` | The employee's own record is inactive, so they cannot change it. | R29 |
| `employee_not_found` | The employee named does not exist. | R1–R4, R7–R9 |
| `change_not_found` | The pending change named does not exist. | R6, R11, R12 |
| `change_already_pending` | This field already has a pending change. | R31 |
| `change_already_decided` | The change is no longer pending. | R33 |
| `record_changed_since_viewed` | Someone changed the record after this user opened it. | R32 |
| `rejection_reason_required` | A rejection must give a reason. | R12 |
| `approval_confirmation_required` | The current value differs from the value when the change was submitted; the approver must confirm. | R13 |
| `change_not_saved` | The change could not be saved with its change-record entry; nothing changed. | R34 |
| `editing_unavailable` | Editing employee information is switched off. | R54 |
| `invalid_mobile_number` | The mobile number is not a valid international number. | R14 |
| `invalid_personal_email` | The personal email is not a well-formed address. | R15 |
| `personal_email_same_as_work_email` | The personal email is the same as the work email. | R15 |
| `address_incomplete` | The address is missing a required part. | R16 |
| `emergency_contact_incomplete` | The emergency contact is missing a name, relationship or phone. | R17 |
| `invalid_emergency_contact_phone` | The emergency contact's phone number is not valid. | R17 |
| `bank_details_incomplete` | The bank details are missing a required part. | R18 |
| `invalid_bank_details_format` | An account number or bank/branch code is in the wrong format. | R18 |
| `reporting_manager_not_active` | The proposed manager is not an active employee. | R19 |
| `reporting_manager_creates_cycle` | The proposed manager is the employee or reports to them. | R19 |
| `invalid_salary_amount` | The salary is not a positive amount. | R20 |
| `unknown_department` | The department is not one of the organisation's departments. | R21 |
| `unknown_designation` | The designation is not one of the organisation's designations. | R21 |

## 3. User-visible behaviour

| Situation | What the user experiences | R-ID |
|---|---|---|
| first use, nothing exists yet | The employee is told they have no submitted changes and can start one. An HR user with nothing awaiting a decision is told so. | R5, R10 |
| search or filter finds nothing | N/A — because this unit has no search; finding employees is in AXW-C001-U001. | — |
| only part of the information could be loaded | The editable details show, and the change record is marked unavailable. Editing stays possible. | R22, R24 |
| waiting for a slow response | The user is told the action is in progress and cannot send it again. | R36 |
| user not allowed to see this | Told they don't have access. No values are shown, and fields they can't change aren't offered for editing. | R25, R39 |
| connection lost while viewing | What was shown stays, marked as possibly out of date. Saving is not possible until the connection is back. | R32 |
| connection lost while submitting | Typed values are kept. The user is told the change was not confirmed, and resending does not duplicate it. | R37, R35 |
| session expires mid-task | Typed values are kept after the same user signs back in. | R37 |
| user leaves with unsaved changes | Warned before leaving; they can stay or discard. | R38 |

## 4. Non-functional requirements

| R-ID | Category | Requirement | Source |
|---|---|---|---|
| R40 | availability | 99% during business hours, measured monthly. The only dependency is sign-in, whose availability is not stated (Q4). | A10 |
| R41 | latency | Save, submit, approve, reject, withdraw: p95 ≤ 2 s, p99 ≤ 4 s. Loading the pending list or a change record: p95 ≤ 3 s, p99 ≤ 6 s. | A9 |
| R42 | throughput | No figure given. Assumed to be within ordinary office-application load. Headcount and peak to be confirmed. | A11, Q5 |
| R43 | surge | At 2× peak, responses may slow beyond R41. No save, decision or change-record entry is ever dropped. | A11 |
| R44 | idempotency | Save, submit, approve, reject and withdraw: repeating the same request has no further effect. The repeat is identified by the submitter and a request identity fixed before the request is sent. | R35, D-skill |
| R45 | concurrency | Two saves to the same record: the second, if based on an older version, is refused (R32). Two decisions, or a decision and a withdrawal, on one change: exactly one succeeds (R33). Two submissions for one field: exactly one becomes pending (R31). | A19 |
| R46 | rate limits | N/A — because no limit was asked for and volumes are low (A11). To be revisited if Q5 shows high volume. | A11 |
| R47 | authorization | Employee: own record; mobile and email take effect directly; address and emergency contact are submitted or withdrawn; can view own history. HR user: every field on every record, including inactive ones; can decide pending changes except their own or ones to their own record. Manager: no rights beyond those of an employee. No other role may edit. Service credentials: none in this unit. | IN-7, IN-8, D5, D8, D11, D15 |
| R48 | tenant isolation | N/A — because the system serves one organisation. | A12 |
| R49 | audit | The change record per R22. Entries are never changed or deleted by any user. Retention is not stated (Q2). | IN-6, D14, Q2 |
| R50 | observability | Metrics: counts of saves, submissions, approvals, rejections and withdrawals; refusals by reason code; age of the oldest pending change. Log fields: change id, employee id, acting user, role, field name, reason code, never field values. Traces: one span per user action. Test: "my address change never applied" is diagnosable from change id + state + reason code. | quality.md |
| R51 | data classification | Mobile, personal email and residential address: personal. Emergency contact: personal data of a third party. Department, designation and reporting manager: personal (work). Salary and bank details: personal, financial and confidential. No special-category fields. Salary and bank values never appear in logs and are masked in an employee's own history (R24). | intent §6, D14 |
| R52 | retention and deletion | The change record is kept for a period to be set by Compliance. The erasure path is to be confirmed (Q2). This unit deletes no employee data. | Q2 |
| R53 | migration and backfill | N/A — because existing employee records are edited in place, and the change record starts empty at go-live with no backfilled history. | A14 |
| R54 | feature flag | When off, every edit, submission, decision and withdrawal is refused with `editing_unavailable`. Pending changes are kept unchanged and can be decided once it is back on. | A15 |
| R55 | accessibility (UI units) | WCAG 2.2 Level AA for every screen in this unit. | A16, C4 |

## 5. Data

| Entity | Owns or Reads | Notes |
|---|---|---|
| Employee record: the fields listed in this unit | Owns (changes) | Creating records is out of scope (OUT-2). The owner of record creation is Q4. AXW-C001-U001 reads it. |
| Active/inactive status | Reads | Deactivation is out of scope (OUT-3). The owner is not decided (Q4). |
| Pending change | Owns | States: pending → approved, rejected or withdrawn. |
| Change record | Owns | Append-only. |
| Departments, designations | Reads | Assumed to exist in this system (A4). |
| User identity and role (employee, HR user) | Reads | From sign-in. |

## 6. Dependencies

| Dependency | What we need from it | How reliable it is | Source |
|---|---|---|---|
| Sign-in | Who the user is and whether they are an HR user | Not stated (Q4) | intent §6 |
| Departments and designations lists | Valid values for R21 | Held in this system | A4 |
| Payroll and other consumers of salary and bank details | Whether a change must reach them | Not stated (Q3) | intent §6 |

## 7. Constraints

| # | Constraint | Type | Source | Confirmed? |
|---|---|---|---|---|
| C1 | Personal and financial data (salary, bank details, third-party emergency contact) is subject to the applicable data-protection law. Which law applies depends on the countries employees are in. | legal | intent §6; BA | No (Compliance/Legal, Q2) |
| C2 | Mobile and email take effect without approval. Address, emergency contact, department, designation, reporting manager, bank details and salary need HR approval. | policy | D6, D11 | Yes |
| C3 | Only an HR user may update an inactive employee's record. | policy | D8 | Yes |
| C4 | Accessibility standard for employee-facing screens | policy | A16 | No (Product owner) |

## 8. Assumptions

| # | Assumption | If it's wrong | Owner to confirm |
|---|---|---|---|
| A1 | Mobile, emergency-contact phone and postal code use international formats valid for any country. | Validation must follow specific country rules (R14, R16, R17) | HR Ops lead (Q1) |
| A2 | Bank details follow one country's format. | Multi-country bank formats are needed (R18) | HR Ops lead (Q1) |
| A3 | Salary is in one organisation currency. | Currency must be captured per employee (R20) | HR Ops lead (Q1) |
| A4 | Departments and designations already exist as lists in this system. | A new dependency, or this unit must maintain the lists | HR Ops lead |
| A5 | "Reports to them" includes indirect reports. | R19 is narrowed to direct reports | Shiny Priscilla J |
| A6 | An HR user cannot decide a change to their own record, even if someone else submitted it. | R28 is narrowed | Shiny Priscilla J |
| A7 | One pending change per field also applies to HR-only fields. | Queued changes per field are needed | Shiny Priscilla J |
| A8 | Pending changes do not expire. | An expiry rule and its time-boundary cases are needed | Shiny Priscilla J |
| A9 | Latency targets in R41 | R41 changes | Product owner |
| A10 | 99% availability during business hours | R40 changes | Product owner |
| A11 | Volume is within ordinary office-application load. | Throughput, surge and rate limits need figures | HR Ops lead (Q5) |
| A12 | A single organisation with no tenants | Tenant isolation must be specified | Product owner |
| A13 | Employee records, status and lists are all held in this system; sign-in is the only outside dependency. | Dependency failure cases are needed | Architect / Product owner |
| A14 | No historical change record is backfilled. | A migration is needed | Product owner |
| A15 | A feature flag exists, and turning it off means falling back to the email process. | R54 is withdrawn | Product owner |
| A16 | WCAG 2.2 AA | R55 changes | Product owner |
| A17 | Typed values survive session expiry for the same user. | R37 is narrowed to connection loss | Product owner |
| A18 | An approved change takes effect at once, with no future-dated changes; a mistake is fixed by making a new change. | Effective dating and reversal are needed | HR Ops lead |
| A19 | A stale save is refused rather than merged. | R32 changes | Product owner |
| A20 | Saving a value identical to the current one is a silent no-op. | A reason code is needed | Product owner |
| A21 | Times are recorded in one reference zone and shown in the viewer's local time. | Display rules change | Product owner |

## 9. Decisions

| # | Decision | Decided by | Date | Carried from intent? |
|---|---|---|---|---|
| D1 | New unit U002; U001 keeps listing, searching and viewing. | Shiny Priscilla J | 2026-11-20 | Yes |
| D2 | "Manage" means editing existing records only; adding and deactivating are out of scope. | Shiny Priscilla J | 2026-11-20 | Yes |
| D3 | Employees may update mobile, personal email, residential address and emergency contact. | Shiny Priscilla J | 2026-11-20 | Yes |
| D4 | Only HR may update department, designation, reporting manager, salary and bank details. | Shiny Priscilla J | 2026-11-20 | Yes |
| D5 | Managers view reporting employees but cannot modify them. | Shiny Priscilla J | 2026-11-20 | Yes |
| D6 | No approval: mobile, personal email. Approval needed: address, emergency contact, department, designation, reporting manager, bank details. | Shiny Priscilla J | 2026-11-20 | Yes |
| D7 | Rejected changes are not applied and the employee receives the reason; approved changes take effect immediately. | Shiny Priscilla J | 2026-11-20 | Yes |
| D8 | Only HR may update inactive employee records. | Shiny Priscilla J | 2026-11-20 | Yes |
| D9 | The problem: requests by email to HR cause delays and manual effort. | Shiny Priscilla J | 2026-11-20 | Yes |
| D10 | Success: reduced HR involvement, faster turnaround, fewer manual errors, complete audit tracking. | Shiny Priscilla J | 2026-11-20 | Yes |
| D11 | HR changes to approval-required fields, and salary, need approval by a different HR user; nobody approves their own change. | ArunVijay | 2026-11-20 | No |
| D12 | One pending change per field. An employee may withdraw but not edit a pending change. A pending change survives the employee becoming inactive. An approver is warned if the field changed since submission. | ArunVijay | 2026-11-20 | No |
| D13 | Validation rules as in R14–R21. An emergency contact is one contact. Bank details are checked for format only. | ArunVijay | 2026-11-20 | No |
| D14 | HR sees the full change record. An employee sees their own history with salary and bank masked. The data-protection law is recorded as a constraint to confirm. | ArunVijay | 2026-11-20 | No |
| D15 | The outcome is shown inside the system, with no email in this unit. HR may change mobile and email directly. No roles other than employee and HR edit. | ArunVijay | 2026-11-20 | No |

## 10. Glossary additions

| Term | Meaning in this unit |
|---|---|
| Withdrawn change | A pending change the employee cancelled before a decision; it is never applied. |
| Submitter | The user who submitted a pending change. |
| Decision | Approving or rejecting a pending change. |
| Masked | Shown in a way that hides the actual value. |
| Reason code | The stable name of an outcome the user is refused with. |

## 11. Open questions

| # | Question | Blocking? | Owner | Answer |
|---|---|---|---|---|
| Q1 | Which countries' formats apply to mobile, postal code and bank details, and which currency applies to salary? | No (A1–A3) | HR Ops lead | |
| Q2 | Which data-protection law applies, how long is the change record kept, and what is the erasure path? | No (C1) | Compliance / Legal | |
| Q3 | Must salary or bank-detail changes reach payroll or any other system? | No, unless yes: it adds scope | Payroll owner | |
| Q4 | Who owns creating employee records and their active/inactive status, and how reliable is sign-in? | No | Product owner | |
| Q5 | What are the employee headcount and peak change volume (e.g. after appraisals)? | No (A11) | HR Ops lead | |
| Q6 | Should IN-3 be widened to cover HR changing address, emergency contact, mobile and email (R8, R9 rest on D15 and the glossary)? | No | Shiny Priscilla J | |

## 12. Coverage

| Intent item | Covered by |
|---|---|
| O1 | R1, R2, R3, R4, R9 |
| O2 | R1–R5, R10, R11, R37 |
| O3 | R14–R21, R32, R36, R38 |
| O4 | R22, R23, R24, R34, R35 |
| O5 | R3, R4, R6–R8, R10–R13, R25–R31, R33, R39 |
| IN-1 | R1, R2, R36, R37, R38 |
| IN-2 | R3, R4, R5, R6, R31, R36, R37, R38 |
| IN-3 | R7, R8, R9, R31 |
| IN-4 | R5, R10, R11, R12, R13, R27, R33 |
| IN-5 | R14–R21, R32 |
| IN-6 | R22, R23, R24, R34, R35 |
| IN-7 | R25, R26, R27, R28, R39 |
| IN-8 | R29, R30 |

R40–R55 are NFRs that apply across the unit.
Every O and IN item has at least one R-ID.

## 13. Change log

| Date | R-ID | Change | Why | By |
|---|---|---|---|---|
