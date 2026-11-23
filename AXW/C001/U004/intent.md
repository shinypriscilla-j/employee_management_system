# Intent — Employee Profile View

| | |
|---|---|
| **Project** | AXW — axworks3 |
| **Capability** | AXW-C001 — Features |
| **Unit** | AXW-C001-U004 |
| **Requested by** | Business stakeholders (HR, managers, employees); clarifications given by Shiny Priscilla J |
| **Agreed on** | 2026-11-23 |

## 1. Summary
This unit lets a user select an employee in the Employee List and open a profile showing who that employee is, who they report to and who reports to them. The profile shows directory information only: name, employee ID, designation, department, contact details, the employee's immediate manager and their direct reports, with each related employee shown by name and designation. Users can open only the profiles of employees they are already permitted to see in the Employee List: HR users all active employees, managers the employees within their scope, and employees their own profile. Today, users can find an employee in the list but cannot easily see their details or reporting relationships; identifying an employee's manager takes a few minutes. The target is under 30 seconds. The profile is view only and gets its data from an existing employee-details service, `GET /employees/{id}`.

## 2. Background
Unit AXW-C001-U003 (employee-list-search-filter) gives HR users, managers and employees a list of active employees that they can search and filter. That list shows name, employee ID, department, role, work email address and employment status, and leaves a separate detail view for an individual employee out of its scope.

The stakeholders asked to "enable users to view detailed employee information by selecting an employee from the Employee List." They stated that "the profile view should present relevant employee details along with reporting relationships, including the employee's manager and direct reports," and that the work "extends the Employee List feature by providing access to detailed employee information and organizational context." They named `GET /employees/{id}` as the source of employee data.

Unit AXW-C001-U002 (update-employee-information) handles editing employee records. This unit only displays information.

## 3. Problem
- **Who is affected:** HR users, managers and employees who use the Employee List and need to know more about an employee than the list shows.
- **What goes wrong today:** Users can find an employee in the list but cannot easily view that employee's details or reporting relationships: who the employee reports to and who reports to them.
- **Impact:** Users spend time working out an employee's place in the organisation; identifying an employee's manager takes a few minutes.
- **Evidence:** Stakeholders state that identifying an employee's manager currently takes "a few minutes". No other figures were given.

## 4. Outcomes

| ID | Outcome | How we'll measure it | Baseline today | Target |
|---|---|---|---|---|
| O1 | Users quickly understand who an employee is, who they report to and who reports to them | Time for a user to identify an employee's manager | A few minutes | Under 30 seconds |
| O2 | Users open only the profiles of employees they are permitted to see | Number of occasions a user opens a profile of an employee outside their permitted set (HR: all active employees; manager: employees within their scope; employee: their own profile) | Not stated | None |

## 5. Scope

### In
| # | This unit covers | Serves |
|---|---|---|
| IN-1 | A user selects an employee in the Employee List and opens that employee's profile | O1 |
| IN-2 | The profile shows the employee's name, employee ID, designation, department and contact details | O1 |
| IN-3 | The profile shows the employee's immediate manager, by name and designation | O1 |
| IN-4 | The profile shows the employee's direct reports, each by name and designation | O1 |
| IN-5 | Inactive employees are never shown as the manager or as a direct report on a profile | O1 |
| IN-6 | An HR user can open the profile of any active employee; a manager can open the profiles of employees within their scope; an employee can open only their own profile | O2 |
| IN-7 | A profile can never be opened for an employee outside the viewer's permitted set | O2 |
| IN-8 | Profile information is obtained from the existing `GET /employees/{id}` service | O1 |

### Out
| # | Excluded | Why | Where it goes instead |
|---|---|---|---|
| OUT-1 | Salary, bank details and personal information on the profile | Decided: the profile shows directory information only | Not planned |
| OUT-2 | Reporting levels beyond the immediate manager and direct reports (for example the manager's manager, or indirect reports) | Decided: one level up and one level down only | Not planned |
| OUT-3 | Showing inactive employees, either as the profile subject or as a related employee | Decided: inactive employees are not displayed | Not planned |
| OUT-4 | Building or changing the `GET /employees/{id}` service | Decided: this unit consumes the existing service | Not planned in this unit |
| OUT-5 | Listing, searching and filtering employees | Separate need, owned by another unit | AXW-C001-U003 (employee-list-search-filter) |
| OUT-6 | Editing employee information from the profile | Separate need, owned by another unit | AXW-C001-U002 (update-employee-information) |
| OUT-7 | Opening profiles of employees outside the viewer's Employee List scope, including a manager's or direct report's profile the viewer is not permitted to see | Decided: profile access matches Employee List access | Not planned |

## 6. For requirements
Everything the stakeholders said that `ba-unit-requirements` will need, written out in full. It isn't analysed
here. Where the stakeholders said nothing, write **"Not stated."** The requirements skill will ask about it.

| Topic | What the stakeholders said |
|---|---|
| What users need to do (each action) | Select an employee in the Employee List to open their profile; view the employee's details, immediate manager and direct reports. Gaps — Not stated: whether a user can open a related employee's profile by selecting the manager or a direct report (when that employee is within the viewer's permitted set); how the user returns to the Employee List; what is shown while the profile is loading. |
| Who uses it, and who must not | HR users: profiles of all active employees. Managers: profiles of employees within their scope. Employees: their own profile only. No one may open a profile outside their permitted set. Gaps — Not stated: whether a manager's scope means direct reports only or also indirect reports (also open in AXW-C001-U003); whether a manager can open their own profile; whether any other role (e.g. administrator) uses the profile. |
| Business rules | Only the immediate manager and direct reports are shown; related employees are shown by name and designation; inactive employees are not displayed. Gaps — Not stated: what is shown when an employee has no manager (for example the top of the organisation) or no direct reports; what is shown when the manager is inactive; what happens when a user tries to open a profile outside their permitted set (for example by a direct link); what happens if an employee becomes inactive or changes manager while their profile is open; the order in which direct reports are listed and whether a long list is limited. |
| Information involved, and how sensitive it is (personal, salary, health…) | Name, employee ID, designation, department, contact details, manager (name and designation), direct reports (name and designation). Directory information only; salary, bank details and personal information are excluded. Gaps — Not stated: exactly which contact details are shown (for example work email address, work phone); how "designation" relates to "role" shown in the Employee List. |
| Volumes and peaks (with who gave the figure) | Not stated for this unit. The Employee List holds about 2,000 active employees today, with an expected peak of about 5,000 (Shiny Priscilla J, 2026-11-23, given for AXW-C001-U003). Largest expected number of direct reports for one manager: Not stated. |
| Other systems or processes it depends on, and how reliable they are | Depends on the Employee List (AXW-C001-U003) as the entry point, and on the existing `GET /employees/{id}` service, which the stakeholders state provides the required employee details. Gaps — Not stated: whether that service returns manager and direct-report information (including designation and active status) or these come from elsewhere; whether the service enforces the viewer's permitted set itself; its reliability and what the profile shows when it fails or returns no employee. |
| Money movement, or changes that can't be undone | None; the profile is view only. |
| Constraints: legal, policy, deadline, technical | Technical: employee data comes from the existing `GET /employees/{id}` service. Legal, policy and deadline: Not stated. |
| Quality expectations: speed, availability, security, accessibility | Identify an employee's manager in under 30 seconds; users open only profiles within their permitted set. Time for a profile to load: Not stated. Availability and accessibility: Not stated. |

## 7. Glossary
| Term | Meaning in this unit |
|---|---|
| Employee | A person whose record is held in the system; as a user, can open only their own profile |
| HR user | A member of HR; can open the profile of any active employee |
| Manager | An employee to whom other employees report; can open the profiles of employees within their scope |
| Employee List | The searchable, filterable list of active employees provided by unit AXW-C001-U003, from which a profile is opened |
| Employee profile | The view-only page showing one employee's directory information, immediate manager and direct reports |
| Permitted set | The employees whose profiles a user may open, the same as the employees they may see in the Employee List: all active employees for HR, employees within their scope for a manager, their own record for an employee |
| Scope (of a manager) | The employees a manager may see in the Employee List: the active employees who report to them |
| Directory information | The non-sensitive details shown on a profile: name, employee ID, designation, department and contact details |
| Contact details | The ways to contact an employee shown on the profile, excluding personal information; exact fields not yet stated |
| Designation | The employee's job title |
| Department | The organisational unit an employee belongs to |
| Employee ID | The identifier that uniquely identifies an employee |
| Immediate manager | The employee to whom the profile's employee directly reports |
| Direct report | An employee who reports directly to the profile's employee |
| Related employee | The immediate manager or a direct report shown on a profile |
| Active / inactive employee | An inactive employee is one no longer active in the organisation; inactive employees are not displayed on profiles |
| Sensitive information | Salary, bank details and personal information; never shown on the profile |
| `GET /employees/{id}` | The existing service that returns an employee's details by employee ID, used as the profile's data source |

## 8. Decisions made during clarification
| Question | Answer | Answered by (name or role) | Date |
|---|---|---|---|
| What problem does this solve, and how is success measured? | Users can find employees in the list but can't easily view their details or reporting relationships. The profile lets them quickly understand who an employee is, who they report to and who reports to them. Success: time to identify an employee's manager reduced from a few minutes to under 30 seconds. | Shiny Priscilla J | 2026-11-23 |
| Whose profiles can each role open? | Only employees the viewer is already authorised to see in the Employee List: HR all active employees, managers employees within their scope, employees their own profile. Profiles cannot be opened outside the viewer's permitted scope. | Shiny Priscilla J | 2026-11-23 |
| Which details appear on the profile? | Basic directory information: name, employee ID, designation, department, contact details, manager and direct reports. Salary, bank details and personal information are not included. | Shiny Priscilla J | 2026-11-23 |
| How much of the reporting structure is shown? | Only the immediate manager and direct reports, each shown by name and designation. Inactive employees are not displayed. | Shiny Priscilla J | 2026-11-23 |
| Does `GET /employees/{id}` exist, or does this unit build it? | Treat it as existing and providing the required employee details. This unit consumes the service and displays the profile; building it is out of scope. | Shiny Priscilla J | 2026-11-23 |
| Should this be a new unit or extend an existing one? | New unit. AXW-C001-U003 left the detail view out of its scope; this unit provides it. | Derived from the request and AXW-C001-U003's scope (Business Analyst) | 2026-11-23 |
