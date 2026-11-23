# Requirements — Employee List, Search and Filter

| | |
|---|---|
| **Unit** | AXW-C001-U003 |
| **Parent feature** | AXW-C001 — Features |
| **Surface** | UI + API |
| **Status** | Ready |
| **Approved by** | ArunVijay-VS, 2026-11-23 |

Non-functional and UX behaviour follow the project quality baseline unless §5 says otherwise.

## 1. Story
As an **HR user, manager or employee**, I want to **view, search and filter the active employees I am permitted to see**, so that **I can find a specific employee quickly without relying on spreadsheets or HR**.

Composed from the intent — confirmed by ArunVijay-VS, 2026-11-23.

## 2. Field rules
This unit creates or changes no employee data; it is view only (intent §6).
The rows below are the inputs a user supplies to the list.

| Field | Rule | Who may change it |
|---|---|---|
| Search text | Optional. Matched against employee name and employee ID as a partial, case-insensitive match (Q2). Matched literally; no wildcards or approximate matching (A7). Empty or spaces only means no search (A6). | Any user, over their own permitted set. Takes effect when the search runs. |
| Department filter | Optional. One department chosen from the departments present in the user's permitted set, plus "Not assigned" (Q2, Q5, A2). | Any user, over their own permitted set. Takes effect when applied. |
| Role filter | Optional. One role chosen from the roles present in the user's permitted set, plus "Not assigned" (Q2, Q5, A2). | Any user, over their own permitted set. Takes effect when applied. |
| Sort | One of: name, employee ID, department, role; ascending or descending. Default: name A–Z (Q4). | Any user. Takes effect when chosen. |
| Page | 50 entries per page (Q4). Must be within the pages of the current result (A3). | Any user. Takes effect when chosen. |

**Permitted set** (intent §8, Q1): HR user — all active employees. Manager — their active direct reports plus their own record. Employee — their own record only. A user with more than one role sees the combined set.

## 3. Acceptance criteria

### Happy path
| R-ID | Given | When | Then |
|---|---|---|---|
| R1 | Priya is an HR user; 2,000 employees are active and 140 are inactive | Priya opens the employee list | She sees the 2,000 active employees only, sorted by name A–Z, 50 per page (page 1 of 40) |
| R2 | Active employee Arjun Mehta, E10234, Finance, Accountant, arjun.mehta@company.example | Any user whose permitted set includes Arjun views the list | Arjun's entry shows: Arjun Mehta · E10234 · Finance · Accountant · arjun.mehta@company.example · Active |
| R3 | Ravi is a manager with 6 active direct reports, 1 inactive direct report, and 12 indirect reports | Ravi opens the employee list | He sees 7 entries: his 6 active direct reports and his own record; no indirect reports and no inactive report |
| R4 | Meena is a manager with no active direct reports | Meena opens the employee list | She sees 1 entry: her own record |
| R5 | Arjun is an employee with no HR or manager role | Arjun opens the employee list | He sees 1 entry: his own record |
| R6 | Lakshmi is both an HR user and a manager of 4 people | Lakshmi opens the employee list | She sees all active employees (the combined set of both roles) |
| R7 | Priya's list includes "Priscilla Jacob" and "Prithvi Rao" | Priya searches "pri" | Both appear; "PRI" gives the same result |
| R8 | Priya's list includes E10234 and E21023 | Priya searches "1023" | Both appear |
| R9 | Priya's list includes 85 active employees in Finance | Priya filters by department "Finance" | Exactly those 85 appear |
| R10 | Priya's list includes 12 active employees with the role "Accountant" | Priya filters by role "Accountant" | Exactly those 12 appear |
| R11 | Of the Finance Accountants, only "Arjun Mehta" has a name containing "arj" | Priya searches "arj" and filters department "Finance" and role "Accountant" | Only Arjun Mehta appears; every entry shown matches all three conditions |
| R12 | Priya's list is sorted by name A–Z | Priya sorts by department descending | Entries are reordered by department Z–A |
| R13 | Priya's result has 2,000 entries | Priya moves to page 2 | She sees entries 51–100 in the current sort order |
| R14 | Priya has searched "arj" with the department filter "Finance" | Priya clears the search and filters | She sees her full permitted set again, sorted by name A–Z, page 1 |
| R15 | 5,000 active employees exist; normal usage | A user runs a search, filter, sort or page change | Results are returned within 2 seconds (A1) |
| R16 | Active employee Kiran Das has no department and no role recorded | Priya views the list, then filters department "Not assigned" | Kiran's department and role show "Not assigned"; the filter returns Kiran |

### Failure cases
| R-ID | Class | Given | When | Then |
|---|---|---|---|---|
| R17 | input | No one in Priya's list has "zzq" in their name or ID | Priya searches "zzq" | She is told no employees match her search, and is offered a way to clear it (`no_matching_employees`); her search and filters stay as she entered them |
| R18 | input | E30456 is active but is not Ravi's direct report | Ravi searches "E30456" | He is told no employees match (`no_matching_employees`); nothing indicates that E30456 exists |
| R19 | input | E10999 is inactive | Priya searches "E10999" | She is told no employees match (`no_matching_employees`); the inactive employee is not shown |
| R20 | input | Priya's list has 2,000 entries | Priya searches "   " (spaces only) | The search is ignored and she sees her full permitted set (A6) |
| R21 | input | Priya's result has 2 pages | Priya asks for page 5 | She is told the page does not exist (`page_out_of_range`); her search, filters and sort are unchanged (A3) |
| R22 | authorization | Sam is signed in but has no employee record and no HR role | Sam opens the employee list | He is told he is not permitted to view the list (`not_permitted_to_view_list`); no entries are shown (A4) |
| R23 | concurrency | Ravi is viewing the list, which shows his direct report Anil | Anil becomes inactive, or moves to another manager | Ravi's current view is not updated live; at his next search, filter, sort, page change or reload, Anil no longer appears (Q5) |
| R24 | dependency | The employee records cannot be read (down or timing out) | Any user opens or searches the list | They are told the list is temporarily unavailable and to try again (`employee_list_unavailable`); no partial or stale list is shown as if it were complete |
| R25 | state | Meena's own employee record is inactive, but she is still signed in | Meena opens the employee list | She is told she is not permitted to view the list (`not_permitted_to_view_list`); no entries are shown (A5) |

**Not applicable:** repetition (view only; repeating a request has no side effects and returns the current list), partial failure (no writes, so nothing can be left half-done), ordering (the unit reads current records; it consumes no events), time (no date windows or time-based rules; "active" means active when the request is made).
**Not decided:** none.

## 4. Reason codes

| Code | Meaning | R-ID |
|---|---|---|
| `no_matching_employees` | Nothing in the user's permitted set matches the search and filters | R17, R18, R19 |
| `page_out_of_range` | The requested page is beyond the end of the current result | R21 |
| `not_permitted_to_view_list` | The user has no permitted set (no qualifying role, or their own record is inactive) | R22, R25 |
| `employee_list_unavailable` | Employee records could not be read; this is not a refusal | R24 |

## 5. Differences from the project baseline
None.

The project baseline does not exist yet.
Until it does, these baseline items are open for this unit too: sign-in and role model, availability, latency at p95/p99 (this unit's 2-second figure comes from the intent, R15), rate limits, observability, tenant isolation, accessibility, data classification of name / employee ID / work email, retention and deletion, and UI situations (first use, slow response, connection lost, session expiry).

## 6. Open questions and assumptions

| # | Item | Blocking? | Owner | Answer |
|---|---|---|---|---|
| Q1 | Which reports a manager sees; whether a manager sees their own record; users with more than one role | No | Shiny Priscilla J | Direct reports only; the manager sees their own record too; more than one role means the combined set — ArunVijay-VS, 2026-11-23 |
| Q2 | How search and filters work | No | Shiny Priscilla J | Name and ID: partial, case-insensitive search. Department and role: pick-list filters. All conditions combine — ArunVijay-VS, 2026-11-23 |
| Q3 | Source of employee records, status and reporting lines; spreadsheet migration | No | Shiny Priscilla J | This system is the source (maintained via U002); migration is out of this unit — ArunVijay-VS, 2026-11-23 |
| Q4 | Default sort, sorting, paging, no-match message, manager with no reports | No | Shiny Priscilla J | Name A–Z; sortable by name, ID, department, role; 50 per page; no-match message with clear; own record only — ArunVijay-VS, 2026-11-23 |
| Q5 | Missing department or role; changes while viewing | No | Shiny Priscilla J | Shown as "Not assigned" and filterable; reflected on next search, filter or reload, not live — ArunVijay-VS, 2026-11-23 |
| Q6 | What does "normal usage" mean: concurrent users and searches per day? | No | Shiny Priscilla J | Open (Assumed: see A1.) |
| Q7 | Target for reducing lookup requests to HR (O3) | No | Shiny Priscilla J | Open |
| Q8 | Does any service or integration (not a person) need to read the list? | No | Shiny Priscilla J | Open (Assumed: none; people only.) |
| A1 | The 2-second target (R15) holds for 95% of requests under normal usage | — | Shiny Priscilla J | |
| A2 | Filter choices are the departments and roles present in the user's own permitted set, so a manager isn't shown other departments (R9, R10, R16) | — | Shiny Priscilla J | |
| A3 | Asking for a page past the end is refused, not silently moved to the last page (R21) | — | Shiny Priscilla J | |
| A4 | A signed-in user with no employee record and no HR role has no permitted set (R22) | — | Shiny Priscilla J | |
| A5 | A user whose own record is inactive has no permitted set, even if a role still lets them sign in (R25) | — | Shiny Priscilla J | |
| A6 | Empty or spaces-only search text is treated as no search (R20) | — | Shiny Priscilla J | |
| A7 | Search text is matched literally; characters such as `%` or `*` are not wildcards (R7, R8) | — | Shiny Priscilla J | |

**Out of this story:** A detail view for an individual employee (OUT-1); inactive employees in the list (OUT-2); a company-wide directory for employees (OUT-3); editing employee information — AXW-C001-U002 (OUT-4); adding or deactivating employees (OUT-5); migrating spreadsheet data into the system (Q3); retiring superseded unit AXW-C001-U001; live updates of an open list (Q5).
