# Intent — Employee List, Search and Filter

| | |
|---|---|
| **Project** | AXW — axworks3 |
| **Capability** | AXW-C001 — Features |
| **Unit** | AXW-C001-U003 |
| **Requested by** | Business stakeholders (HR, managers, employees); clarifications given by Shiny Priscilla J |
| **Agreed on** | 2026-11-23 |

## 1. Summary
This unit gives HR users, managers and employees a list of active employees in the system that they can search and filter, so they can find a specific employee quickly. HR users see all active employees, managers see the active employees who report to them, and each employee sees their own record. The list shows each employee's name, employee ID, department, role, work email address and employment status, and can be searched and filtered by name, employee ID, department and role. Today, finding an employee takes 3–5 minutes using shared spreadsheets or by contacting HR. The target is to find a specific employee within 30 seconds, with search and filter results returned within 2 seconds for up to about 5,000 employees.

## 2. Background
Today, HR teams keep employee information in shared spreadsheets. Employees often contact HR directly when they need to find someone, and managers rely on HR or on lists they maintain by hand to identify the employees who report to them. Finding the right employee "can take several minutes, especially when information is outdated or when only partial details are known."

The stakeholders asked for "a way to view employees", so that users can "see the employees that are available in the system and find the employee they are looking for without much effort." They asked for the list to "support searching and filtering", at a minimum by employee name, employee ID, department and role, to "display relevant information about employees", and to "work efficiently even when there are many employees in the system."

The organisation has about 2,000 active employees today and expects a peak of about 5,000.

Unit AXW-C001-U002 (update-employee-information) handles editing employee records. An earlier unit, AXW-C001-U001 (employee-list), was registered for listing employees but its intent was never written. This unit replaces it and owns listing, searching and filtering employees.

## 3. Problem
- **Who is affected:** HR users, who maintain employee information and answer requests to find people; managers, who need to identify the employees who report to them; employees, who contact HR to find someone.
- **What goes wrong today:** Employee information is held in shared spreadsheets and hand-maintained lists that can be out of date. There is no single place to look up an employee, so people search spreadsheets or ask HR, and searching is hard when only partial details are known.
- **Impact:** Employees and managers lose time looking for people, and HR carries administrative work answering lookup requests and maintaining lists.
- **Evidence:** Stakeholders state that finding an employee typically takes 3–5 minutes using spreadsheets or by contacting HR.

## 4. Outcomes

| ID | Outcome | How we'll measure it | Baseline today | Target |
|---|---|---|---|---|
| O1 | Users find a specific employee quickly | Time from a user starting a search to locating the employee they are looking for | 3–5 minutes (spreadsheets or contacting HR) | Within 30 seconds |
| O2 | Search and filter stay responsive as the workforce grows | Time for a search or filter operation to return results under normal usage, with up to about 5,000 employees in the system | No system today; about 2,000 active employees | Within 2 seconds |
| O3 | Finding employees no longer depends on HR or hand-maintained lists | Number of requests to HR to find an employee or to identify a manager's reporting employees | Requests made routinely; count not stated | Reduced; figure not stated |
| O4 | Each user sees only the employees they are permitted to see | Number of occasions a user sees an employee outside their permitted set (HR: all active employees; manager: active employees who report to them; employee: their own record) | Not stated | None |

## 5. Scope

### In
| # | This unit covers | Serves |
|---|---|---|
| IN-1 | An HR user views a list of all active employees | O1, O3 |
| IN-2 | A manager views a list of the active employees who report to them | O1, O3 |
| IN-3 | An employee views their own record in the list | O3, O4 |
| IN-4 | Each list entry shows employee name, employee ID, department, role, work email address and employment status | O1 |
| IN-5 | A user searches and filters the list they are permitted to see by employee name, employee ID, department and role | O1, O2 |
| IN-6 | Search and filter results are returned within 2 seconds under normal usage with up to about 5,000 employees | O2 |
| IN-7 | Only active employees appear in the list; inactive employees never appear | O4 |
| IN-8 | A user never sees employees outside their permitted set | O4 |

### Out
| # | Excluded | Why | Where it goes instead |
|---|---|---|---|
| OUT-1 | A separate detail view for an individual employee | Decided: this unit is a list only | Not decided |
| OUT-2 | Inactive employees in the list, by default, behind a filter or for any role | Decided: only active employees are in scope | Not planned |
| OUT-3 | Employees seeing other employees (a company-wide directory) | Decided: an employee sees only their own record | Not decided |
| OUT-4 | Editing employee information | Separate need, owned by another unit | AXW-C001-U002 (update-employee-information) |
| OUT-5 | Adding or deactivating employees | Not requested for this unit | Not decided |

## 6. For requirements
Everything the stakeholders said that `ba-unit-requirements` will need, written out in full. It isn't analysed
here. Where the stakeholders said nothing, write **"Not stated."** The requirements skill will ask about it.

| Topic | What the stakeholders said |
|---|---|
| What users need to do (each action) | View the list of employees they are permitted to see; search and filter it by employee name, employee ID, department and role ("at a minimum"); find a specific employee when only partial details are known. Gaps — Not stated: which of the four fields are free-text search and which are filters with fixed choices; whether partial or approximate matching applies to name and employee ID; whether search and filters can be combined; default sort order and whether users can sort; whether the list is paged; what is shown when no employee matches; whether a search can be cleared or reset. |
| Who uses it, and who must not | HR users: all active employees. Managers: the active employees who report to them. Employees: their own record only. Gaps — Not stated: whether "report to them" means direct reports only or also indirect reports; whether a manager also sees their own record; what a manager with no reporting employees sees; whether any other role (e.g. administrator) uses the list. |
| Business rules | Only active employees appear. Each user sees only their permitted set. Gaps — Not stated: what happens when an employee becomes inactive or changes manager while a user is viewing the list; how an employee with missing department or role is shown and filtered. |
| Information involved, and how sensitive it is (personal, salary, health…) | Employee name, employee ID, department, role, work email address, employment status (always "Active" in this list). These are work information; no salary, bank, health or personal contact details are shown. Sensitivity classification of each field: Not stated. |
| Volumes and peaks (with who gave the figure) | About 2,000 active employees today; expected peak about 5,000 employees (Shiny Priscilla J, 2026-11-23). Number of concurrent users and searches per day: Not stated. |
| Other systems or processes it depends on, and how reliable they are | Today: shared spreadsheets maintained by HR and hand-maintained manager lists. Where employee records, active status and reporting lines come from in the system: Not stated. Whether spreadsheet data must be migrated: Not stated. |
| Money movement, or changes that can't be undone | None; the list is view only. |
| Constraints: legal, policy, deadline, technical | Not stated. |
| Quality expectations: speed, availability, security, accessibility | Find a specific employee within 30 seconds of starting a search; search and filter results within 2 seconds under normal usage; work efficiently with up to about 5,000 employees; users see only permitted employees. Meaning of "normal usage": Not stated. Availability and accessibility: Not stated. |

## 7. Glossary
| Term | Meaning in this unit |
|---|---|
| Employee | A person whose record is held in the system; as a user, sees only their own record in the list |
| HR user | A member of HR; sees all active employees in the list |
| Manager | An employee to whom other employees report; sees the active employees who report to them |
| Reporting employee | An employee who reports to a given manager |
| Active employee | An employee whose employment status is Active; only active employees appear in the list |
| Inactive employee | An employee who is no longer active in the organisation; never appears in the list |
| Employee list | The list of active employees a user is permitted to see, one entry per employee |
| Permitted set | The employees a user may see: all active employees for HR, active reporting employees for a manager, their own record for an employee |
| Search | Finding employees in the list by entering employee name or employee ID details |
| Filter | Narrowing the list to employees matching chosen criteria such as department or role |
| Department | The organisational unit an employee belongs to |
| Role | The employee's job role in the organisation |
| Work email address | The employee's organisation-issued email address |
| Employment status | Whether an employee is active or inactive; shown as "Active" in this list |
| Normal usage | Everyday use of the list by HR users, managers and employees; not further defined by stakeholders |

## 8. Decisions made during clarification
| Question | Answer | Answered by (name or role) | Date |
|---|---|---|---|
| Should this reuse the existing employee-list unit (AXW-C001-U001)? | No. Create a new unit and leave U001 as it is. | Shiny Priscilla J | 2026-11-23 |
| Who can use the list, and whom can each see? | HR users see all employees; managers see only employees who report to them; each employee sees their own record. | Shiny Priscilla J | 2026-11-23 |
| Is a per-employee detail view in scope? | No. List only. | Shiny Priscilla J | 2026-11-23 |
| What is the problem today? | HR keeps employee information in shared spreadsheets; employees contact HR to find someone; managers rely on HR or hand-maintained lists to identify reporting employees. Finding someone takes several minutes, especially with outdated or partial information, costing employees and managers time and creating administrative work for HR. | Shiny Priscilla J | 2026-11-23 |
| What are the targets and baseline? | Locate a specific employee within 30 seconds of starting a search; search and filter results within 2 seconds under normal usage. Baseline: 3–5 minutes using spreadsheets or contacting HR. | Shiny Priscilla J | 2026-11-23 |
| What volumes must it handle? | About 2,000 active employees today; expected peak about 5,000. | Shiny Priscilla J | 2026-11-23 |
| Should inactive employees appear? | No. Only active employees are in scope. | Shiny Priscilla J | 2026-11-23 |
| Which columns does the list show? | Employee name, employee ID, department, role, work email address, employment status (Active). | Shiny Priscilla J | 2026-11-23 |
| How does this unit relate to U001? | This unit owns listing, search and filtering. U001 is superseded and can be retired once this unit is created and approved. Listing is not split across both units. | Shiny Priscilla J | 2026-11-23 |
