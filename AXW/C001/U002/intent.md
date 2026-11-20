# Intent — Update Employee Information

| | |
|---|---|
| **Project** | AXW — axworks3 |
| **Capability** | AXW-C001 — Features |
| **Unit** | AXW-C001-U002 |
| **Requested by** | Business stakeholders (employee and HR users); clarifications given by Shiny Priscilla J |
| **Agreed on** | 2026-11-20 |

## 1. Summary
This unit lets employees and HR users edit existing employee records in the system. Employees update their own mobile number and personal email directly, and submit changes to their residential address and emergency contact details for HR approval. Only HR updates department, designation, reporting manager, salary and bank details, and only HR may change the records of inactive employees. Every change is validated before it is saved and recorded with who changed what and when. The aim is to stop employees emailing HR for every update, which today causes delays and manual effort, while keeping sensitive changes under HR control.

## 2. Background
Today, an employee who needs their information changed sends an email to HR, and HR makes the change by hand. The stakeholders describe this as causing "delays and additional manual effort".

The stakeholders asked for "a way for employees and HR users to update employee information in the system", stating that "only authorised users can update employee information", that "the system should validate the information before saving it", and that there must be "a record of all changes so that we know who changed what and when". They noted that some changes need HR approval before they take effect, and that updates to inactive employees must be treated differently.

Their stated objective is "to allow employees to maintain their own information while ensuring appropriate control over sensitive changes."

A separate unit (AXW-C001-U001, employee-list) handles listing, searching and viewing employees. This unit covers only editing existing employee records.

## 3. Problem
- **Who is affected:** Employees who need their information changed, and HR users who receive and carry out those change requests.
- **What goes wrong today:** Every change to employee information is requested by email to HR and applied by HR by hand. Employees cannot maintain their own details.
- **Impact:** Updates are delayed, HR spends manual effort processing requests, and manual handling allows errors in employee records. There is no complete record of who changed what and when.
- **Evidence:** Stakeholders state that requests go "through email to HR", which "causes delays and additional manual effort". No figures were given.

## 4. Outcomes

| ID | Outcome | How we'll measure it | Baseline today | Target |
|---|---|---|---|---|
| O1 | HR is less involved in routine updates to employee information | Number of employee-information change requests that HR must handle (by email or approval) | All changes are requested by email to HR; count not stated | Reduced; figure not stated |
| O2 | Employee information is updated faster | Time from an employee asking for a change to the change being in effect | Not stated | Faster; figure not stated |
| O3 | Fewer errors in employee information caused by manual handling | Number of errors in employee records traced to manual re-entry | Not stated | Fewer; figure not stated |
| O4 | Every change to employee information can be traced | Share of changes to employee information with a record of who changed it, what changed and when | Not stated | 100% (stakeholders: "complete audit tracking") |
| O5 | Sensitive changes stay under HR control | Share of changes to approval-required fields that took effect without HR approval; changes made by anyone not permitted to make them | Not stated | None |

## 5. Scope

### In
| # | This unit covers | Serves |
|---|---|---|
| IN-1 | An employee updates their own mobile number and personal email; the change takes effect immediately without approval | O1, O2 |
| IN-2 | An employee submits a change to their own residential address or emergency contact details, which takes effect only after HR approval | O1, O2, O5 |
| IN-3 | An HR user updates an employee's department, designation, reporting manager, salary and bank details | O5 |
| IN-4 | An HR user reviews a pending change and approves it (applied immediately) or rejects it with a reason (not applied; the employee receives the reason) | O2, O5 |
| IN-5 | Information is validated before it is saved | O3 |
| IN-6 | Every change to employee information is recorded with who changed it, what changed and when | O4 |
| IN-7 | Only authorised users can change employee information: an employee changes only their own permitted fields; a manager cannot modify their reporting employees' information | O5 |
| IN-8 | Only an HR user can update the record of an inactive employee; employees and managers cannot | O5 |

### Out
| # | Excluded | Why | Where it goes instead |
|---|---|---|---|
| OUT-1 | Listing, searching and viewing employees, including a manager viewing their reporting employees | Separate need, already owned by another unit | AXW-C001-U001 (employee-list) |
| OUT-2 | Adding new employees | Stakeholders limited "manage" to editing existing records | Not decided |
| OUT-3 | Deactivating employees | Stakeholders limited "manage" to editing existing records | Not decided |

## 6. For requirements
Everything the stakeholders said that `ba-unit-requirements` will need, written out in full. It isn't analysed
here. Where the stakeholders said nothing, write **"Not stated."** The requirements skill will ask about it.

| Topic | What the stakeholders said |
|---|---|
| What users need to do (each action) | Employee: update own mobile number, personal email, residential address, emergency contact details. HR user: update department, designation, reporting manager, salary, bank details; approve or reject pending changes, giving a reason on rejection; update inactive employee records. Manager: cannot modify reporting employees' information. Gaps — Not stated: whether HR can also edit the employee-editable fields; whether an employee can edit or withdraw a pending change; what happens when a second change is submitted for a field that already has one pending; which details make up "emergency contact details". |
| Who uses it, and who must not | Employees (own record, permitted fields only, active records only); HR users (all listed fields, including inactive records); managers must not modify any employee's information. Not stated: whether any other role (e.g. administrator) can make changes. |
| Business rules | No approval: mobile number, personal email. HR approval required: residential address, emergency contact details, department, designation, reporting manager, bank details. Approved changes take effect immediately. Rejected changes are not applied and the employee receives the rejection reason. Only HR may update inactive employee records. Gaps — Not stated: whether salary changes need approval (salary is HR-only but not in the approval list); whether a change made by an HR user to an approval-required field needs approval by a different HR user; what happens to a pending change if the employee becomes inactive before it is decided; whether an employee is notified when a change is approved. |
| Information involved, and how sensitive it is (personal, salary, health…) | Mobile number, personal email, residential address, emergency contact details (personal contact information, including details of people other than the employee); department, designation, reporting manager (work information); salary and bank details (financial). Stakeholders refer to "sensitive changes" without classifying each field. Validation rules for each field: Not stated. |
| Volumes and peaks (with who gave the figure) | Not stated. |
| Other systems or processes it depends on, and how reliable they are | Today's process is email to HR. Where an employee's active/inactive status comes from: Not stated. How the employee receives a rejection reason (channel): Not stated. Other systems that consume employee information: Not stated. |
| Money movement, or changes that can't be undone | Salary and bank details can be changed by HR. Whether these changes affect payroll or payments: Not stated. Whether a change can be reversed once applied: Not stated. |
| Constraints: legal, policy, deadline, technical | Not stated. |
| Quality expectations: speed, availability, security, accessibility | Only authorised users can update employee information; information is validated before saving; every change is recorded with who, what and when. Who may view the change record, and how long it is kept: Not stated. Speed, availability, accessibility: Not stated. |

## 7. Glossary
| Term | Meaning in this unit |
|---|---|
| Employee | A person whose record is held in the system and who can update certain fields of their own record |
| HR user | A member of HR who can update any listed field, approve or reject pending changes, and update inactive employee records |
| Manager | An employee to whom other employees report; can view but not modify their reporting employees' information (viewing is in AXW-C001-U001) |
| Employee record | The stored information about one employee that this unit edits |
| Active / inactive employee | An inactive employee is one no longer active in the organisation; only HR may update their record |
| Immediate change | A change to mobile number or personal email, which takes effect when saved, with no approval |
| Approval-required change | A change to residential address, emergency contact details, department, designation, reporting manager or bank details, which takes effect only after HR approves it |
| Pending change | An approval-required change that has been submitted but not yet approved or rejected |
| Rejection reason | The explanation an HR user gives when rejecting a pending change, which the employee receives |
| Change record | The record kept for every change to employee information, showing who changed it, what changed and when |
| Validation | Checking information is acceptable before it is saved |

## 8. Decisions made during clarification
| Question | Answer | Answered by (name or role) | Date |
|---|---|---|---|
| Is this a new unit, or does it extend the employee-list unit? | New unit, U002 — Update Employee Information. U001 stays responsible for listing, searching and viewing employees. | Shiny Priscilla J | 2026-11-20 |
| What does "manage employee information" include? | Only editing existing employee records. Adding and deactivating employees are out of scope. | Shiny Priscilla J | 2026-11-20 |
| Which fields can employees update? | Mobile number, personal email, residential address, emergency contact details. | Shiny Priscilla J | 2026-11-20 |
| Which fields can only HR update? | Department, designation, reporting manager, salary, bank details. | Shiny Priscilla J | 2026-11-20 |
| What can managers do? | View their reporting employees; they cannot directly modify their information. | Shiny Priscilla J | 2026-11-20 |
| Which changes need HR approval? | Not needed: mobile number, personal email. Needed: residential address, emergency contact details, department, designation, reporting manager, bank details. | Shiny Priscilla J | 2026-11-20 |
| What happens on rejection and on approval? | Rejected changes are not applied and the employee receives the rejection reason. Approved changes take effect immediately. | Shiny Priscilla J | 2026-11-20 |
| How are inactive employees handled? | Only HR may update inactive employee records; employees and managers may not. | Shiny Priscilla J | 2026-11-20 |
| What is the problem today? | Employees request updates by email to HR, causing delays and additional manual effort. | Shiny Priscilla J | 2026-11-20 |
| How is success measured? | Reduced HR involvement, faster update turnaround, fewer manual errors, complete audit tracking of changes. | Shiny Priscilla J | 2026-11-20 |
