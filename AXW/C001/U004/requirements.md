# Requirements — Employee Profile View

| | |
|---|---|
| **Unit** | AXW-C001-U004 |
| **Parent feature** | AXW-C001 — Features |
| **Surface** | UI + API |
| **Status** | Ready |
| **Approved by** | Shiny Priscilla J, 2026-11-24 |

Non-functional and UX behaviour follow the project quality baseline unless §5 says otherwise.

## 1. Story
As an **HR user, manager or employee**, I want to **open, from the Employee List, the profile of an employee I am permitted to see**, so that **I can quickly understand who they are, who they report to and who reports to them**.

Composed from the intent — confirmed by Shiny Priscilla J, 2026-11-24.

## 2. Field rules
This unit creates or changes no employee data; the profile is view only (intent §6).
Employee data is maintained through AXW-C001-U002.
The rows below are the input that selects a profile and the rules for what the profile shows.

| Field | Rule | Who may change it |
|---|---|---|
| Profile subject | The employee whose profile is opened, identified by employee ID. Must be an active employee in the viewer's permitted set; otherwise the profile is refused (`profile_not_available`) with no sign of whether the employee exists (Q4). | Any user, within their own permitted set. Takes effect when the profile is opened. |
| Directory information | Name, employee ID, designation, department, work email and work phone (intent §8, Q1). Designation is the same value as "role" in the Employee List (Q1). A missing department or designation is shown as "Not assigned", as in the Employee List; a missing work phone is shown as "Not recorded" (A4). Salary, bank details and personal information are never shown (intent OUT-1). | Not changeable here. |
| Immediate manager | Shown by name and designation (intent §8). Shown as "No manager" when the employee has no manager or the manager is inactive (Q3). | Not changeable here. |
| Direct reports | Every active direct report, each by name and designation. Ordered by name A–Z. All are shown, with no paging and no limit (Q5). Inactive direct reports are left out. "No direct reports" is shown when there are none (Q3). | Not changeable here. |
| Related-employee link | The manager or a direct report can be selected to open their profile only when that employee is in the viewer's permitted set. Otherwise their name is shown as plain text (Q2). | Not changeable here. |

**Permitted set** (intent §8: same as the Employee List; defined in AXW-C001-U003 Q1):
- HR user: all active employees.
- Manager: their active direct reports plus their own record.
- Employee: their own record only.
- A user with more than one role: the combined set.

## 3. Acceptance criteria

### Happy path
| R-ID | Given | When | Then |
|---|---|---|---|
| R1 | Priya is an HR user; her Employee List shows active employee Arjun Mehta, E10234 | Priya selects Arjun in the Employee List | Arjun's profile opens |
| R2 | Arjun Mehta, E10234, designation Accountant, department Finance, arjun.mehta@company.example, +44 20 7946 0123 | Priya views Arjun's profile | It shows: Arjun Mehta · E10234 · Accountant · Finance · arjun.mehta@company.example · +44 20 7946 0123; the designation matches the role shown for Arjun in the Employee List; no salary, bank or personal details appear |
| R3 | Arjun reports to Ravi Kumar, Finance Manager, who is active | Priya views Arjun's profile | The manager shows as: Ravi Kumar · Finance Manager |
| R4 | Ravi has 6 active direct reports, including Bala Iyer, Anil Shah and Arjun Mehta | Priya views Ravi's profile | All 6 are listed by name and designation, ordered by name A–Z (Anil Shah, Arjun Mehta, Bala Iyer, …), with no paging |
| R5 | Ravi also has 1 inactive direct report, Deepa Nair | Priya views Ravi's profile | Deepa is not listed; 6 direct reports are shown |
| R6 | Kiran's manager, Suresh Pillai, is inactive | Priya views Kiran's profile | The manager shows as "No manager"; Suresh is not shown |
| R7 | Anita Rao, the CEO, has no manager | Priya views Anita's profile | The manager shows as "No manager" |
| R8 | Arjun has no direct reports | Priya views Arjun's profile | Direct reports shows "No direct reports" |
| R9 | Priya is an HR user | Priya opens the profile of any active employee, for example Anita Rao or Arjun Mehta | The profile opens |
| R10 | Ravi is a manager with 6 active direct reports, including Arjun | Ravi opens Arjun's profile, then his own | Both profiles open |
| R11 | Arjun is an employee with no HR or manager role | Arjun opens his own profile | His profile opens |
| R12 | Lakshmi is both an HR user and a manager of 4 people | Lakshmi opens the profile of an active employee outside her team | The profile opens (the combined set of both roles) |
| R13 | Priya is viewing Arjun's profile; his manager Ravi is in her permitted set | Priya selects Ravi | Ravi's profile opens |
| R14 | Arjun, an employee, is viewing his own profile; his manager Ravi is not in his permitted set | Arjun views the manager section | Ravi Kumar · Finance Manager is shown as plain text and cannot be selected |
| R15 | Priya searched "arj", filtered by department "Finance", went to page 2 and then opened Arjun's profile | Priya chooses "Back to Employee List" | The Employee List shows her search "arj", the "Finance" filter and page 2 as she left them |
| R16 | 5,000 active employees exist; the profile has 50 direct reports (A1); normal usage | A user opens a profile | It is shown within 2 seconds for 95% of requests |

### Failure cases
| R-ID | Class | Given | When | Then |
|---|---|---|---|---|
| R17 | authorization | E30456 is active but is not Ravi's direct report | Ravi opens E30456's profile by a direct link | He is told "Profile not available" (`profile_not_available`); nothing shows whether E30456 exists |
| R18 | authorization | Arjun is an employee with no HR or manager role | Arjun opens Ravi's profile by a direct link | He is told "Profile not available" (`profile_not_available`); no details of Ravi are shown |
| R19 | authorization | Sam is signed in but has no employee record and no HR role | Sam opens any profile, for example E10234 | He is told "Profile not available" (`profile_not_available`); no details are shown (A2) |
| R20 | input | Active employee Kiran Das has no department, designation or work phone recorded | Priya views Kiran's profile | Department and designation show "Not assigned" and work phone shows "Not recorded"; the rest of the profile is shown as normal (A4) |
| R21 | input | No employee E99999 exists | Priya opens E99999's profile by a direct link | She is told "Profile not available" (`profile_not_available`), the same response as R17 |
| R22 | input | — | Priya opens the profile for the malformed ID "abc!!" by a direct link | She is told "Profile not available" (`profile_not_available`), the same response as R17 |
| R23 | concurrency | Priya has Ravi's profile open, listing his direct report Anil | Anil becomes inactive or moves to another manager | The open profile is not updated live; when Priya opens Ravi's profile again or reloads it, Anil is no longer listed |
| R24 | dependency | The employee details cannot be read (down or timing out), including the manager or direct reports | Any user opens a profile | They are told the profile is temporarily unavailable and to try again (`employee_details_unavailable`); no partial profile is shown (A3) |
| R25 | state | E10999 is inactive | Priya opens E10999's profile by a direct link | She is told "Profile not available" (`profile_not_available`), the same response as R17 |
| R26 | state | Meena's own employee record is inactive, but she is still signed in | Meena opens her own profile | She is told "Profile not available" (`profile_not_available`); no details are shown (A2) |
| R27 | state | Priya opened Arjun's profile, showing manager Ravi as a link; Ravi then became inactive | Priya selects Ravi | She is told "Profile not available" (`profile_not_available`) |

**Not applicable:** repetition (view only; repeating a request has no side effects and returns the current profile), partial failure (no writes, so nothing can be left half-done), ordering (the unit reads current records and consumes no events), time (no date windows or time-based rules; "active" means active when the profile is opened).
**Not decided:** none.

## 4. Reason codes

| Code | Meaning | R-ID |
|---|---|---|
| `profile_not_available` | The profile is not in the viewer's permitted set, the employee is inactive, or no such employee exists. These cases cannot be told apart. | R17, R18, R19, R21, R22, R25, R26, R27 |
| `employee_details_unavailable` | The employee details could not be read; this is not a refusal | R24 |

## 5. Differences from the project baseline
None.

The project baseline does not exist yet.
Until it does, these baseline items are open for this unit too: sign-in and role model, availability, latency at p99 (this unit's p95 of 2 seconds comes from the user, R16), rate limits, observability, tenant isolation, accessibility, data classification of name / employee ID / work email / work phone, retention and deletion, and UI situations (loading / slow response, connection lost, session expiry).

## 6. Open questions and assumptions

| # | Item | Blocking? | Owner | Answer |
|---|---|---|---|---|
| Q1 | Which contact details are shown; how designation relates to role | No | Shiny Priscilla J | Work email and work phone; designation is the same value as the list's "role" — Shiny Priscilla J, 2026-11-24 |
| Q2 | Opening a related employee's profile; returning to the Employee List | No | Shiny Priscilla J | Linked only if in the viewer's permitted set, otherwise plain text; "Back to Employee List" keeps the earlier search, filters and page — Shiny Priscilla J, 2026-11-24 |
| Q3 | No manager, inactive manager, no direct reports | No | Shiny Priscilla J | "No manager" for no manager or an inactive one; "No direct reports" — Shiny Priscilla J, 2026-11-24 |
| Q4 | Profile outside the permitted set, inactive or unknown | No | Shiny Priscilla J | One response, "Profile not available"; existence is not revealed — Shiny Priscilla J, 2026-11-24 |
| Q5 | Order and limit of direct reports | No | Shiny Priscilla J | Name A–Z; all shown, no paging or limit — Shiny Priscilla J, 2026-11-24 |
| Q6 | What is the largest number of direct reports for one manager? | No | Shiny Priscilla J | Open (Assumed: 50 — A1.) |
| Q7 | Does any other role (for example administrator), or any service rather than a person, use the profile? | No | Shiny Priscilla J | Open (Assumed: none; HR users, managers and employees only, as in AXW-C001-U003 Q8.) |
| A1 | A manager has at most 50 direct reports, for the R16 load target and for design (R4, R16) | — | Shiny Priscilla J | Provisional, agreed 2026-11-24 |
| A2 | A signed-in user with no employee record and no HR role, or whose own record is inactive, has no permitted set, as in AXW-C001-U003 A4/A5 (R19, R26) | — | Shiny Priscilla J | |
| A3 | If any part of the profile (details, manager or direct reports) cannot be read, the whole profile is unavailable; no partial profile is shown (R24) | — | Shiny Priscilla J | |
| A4 | A missing department or designation shows "Not assigned", as in the Employee List; a missing work phone shows "Not recorded" (R20) | — | Shiny Priscilla J | |

**Out of this story:** salary, bank details and personal information (OUT-1); reporting levels beyond the immediate manager and direct reports (OUT-2); showing inactive employees (OUT-3); building or changing the employee-details service (OUT-4); listing, searching and filtering — AXW-C001-U003 (OUT-5); editing from the profile — AXW-C001-U002 (OUT-6); profiles outside the viewer's permitted set (OUT-7); live updates of an open profile.
