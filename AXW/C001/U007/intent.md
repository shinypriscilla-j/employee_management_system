# Intent – Employee Leave Request Cancellation

| Field | Details |
| --- | --- |
| **Unit** | `AXW-C001-U007 – leave-request-cancellation` |
| **Capability** | `AXW-C001` |
| **Jira Story** | None yet |
| **Lane** | `business-change` |
| **Type** | `API + UI` |

## User Story

**As an** employee, **I want** to cancel my own leave requests from the My Leave page, **so that** I can withdraw leave when my plans change without emailing HR and waiting for them to cancel it manually.

## Scope

| ID | In Scope |
| --- | --- |
| IN-01 | An employee can cancel their own leave request that is pending approval, from the My Leave page. |
| IN-02 | An employee can cancel their own approved leave request, from the My Leave page, up to and including the day before the first day of leave. |
| IN-03 | An employee can give a reason when cancelling. The reason is optional. |
| IN-04 | When an approved request is cancelled, the days in that request are returned to the employee's leave balance straight away. |
| IN-05 | When a request is cancelled, the employee's line manager is sent an email that includes the employee's reason when one was given. |
| IN-06 | An employee cannot cancel an approved request on or after its first day of leave; that cancellation goes to HR. |

| ID | Out of Scope |
| --- | --- |
| OUT-01 | Changes to how managers approve or reject leave requests. |
| OUT-02 | HR cancelling a leave request on an employee's behalf. The back office already does this. |
| OUT-03 | Employees cancelling leave that has already started. HR handles these. |
| OUT-04 | Cancelling some of the days in a request. Only the whole request can be cancelled. |
| OUT-05 | In-app notifications. The manager is told by email only. |
| OUT-06 | Cancelling a request in any status other than pending or approved, such as rejected or already cancelled. |

## Business Rules

| ID | Rule | Example |
| --- | --- | --- |
| BR-01 | Employees can cancel only their own leave requests. | Priya can cancel her own request. She cannot cancel a request belonging to a colleague. |
| BR-02 | A pending request can be cancelled at any time before a decision is made on it. | Priya submits a request for 15–17 Dec. It is still waiting for approval, and she cancels it on 1 Dec. The cancellation goes through. |
| BR-03 | An approved request can be cancelled up to and including the day before its first day of leave. From the first day of leave onwards, the employee cannot cancel it and must contact HR. | Approved leave runs 10–12 Dec. Cancelling on 9 Dec is allowed. Cancelling on 10 Dec is not, and the employee is told to contact HR. |
| BR-04 | Cancellation applies to the whole request only. | A request covers Mon–Fri. The employee can cancel all five days but cannot cancel only Friday. |
| BR-05 | Days come off the leave balance only when a request is approved, so cancelling a pending request does not change the balance. | Balance is 12 days. A pending 3-day request is cancelled. The balance stays at 12. |
| BR-06 | Cancelling an approved request returns all of its days to the leave balance straight away. | Balance was 12. It dropped to 9 when a 3-day request was approved. The request is cancelled and the balance is back to 12 straight away. |
| BR-07 | A cancellation reason is optional. Cancelling without a reason is allowed. | The employee cancels and leaves the reason empty. The cancellation goes through. |
| BR-08 | Every cancellation sends an email to the employee's line manager. The email includes the reason when one was given. | Priya cancels with the reason "Trip postponed". Her line manager gets an email saying the request was cancelled, with the reason "Trip postponed". |

## Dependencies

| Depends On | Why It Is Needed |
| --- | --- |
| Existing leave request submission and approval process | There must be leave requests, each with a status of pending or approved, for an employee to cancel. |
| Existing leave balance | Days from a cancelled approved request are returned to it. |
| Existing My Leave page | This is where the employee starts a cancellation. |
| Employee–line manager relationship | Needed to know which manager to email. |
| Notification Service (Platform team) | Sends the cancellation email to the line manager. |

## Decisions

| ID | Question | Decision | By | Date |
| --- | --- | --- | --- | --- |
| DEC-01 | What counts as "the leave has started" for an approved request? | An employee can cancel up to the day before the first day of leave. From the first day onwards, cancellation goes to HR. This gives a clear cut-off and avoids disputes about cancelling part-way through a day. | ArunVijay-VS | 2026-11-26 |
| DEC-02 | Can an employee cancel some of the days in a multi-day request? | No. Only the whole request can be cancelled. Partial cancellation would need date-splitting and partial balance changes, so it is left for a possible later unit. | ArunVijay-VS | 2026-11-26 |
| DEC-03 | Does a pending request hold days from the leave balance? | No. Days come off the balance only on approval, so only cancelling an approved request returns days. | ArunVijay-VS | 2026-11-26 |
| DEC-04 | Which manager is told about a cancellation, how, and is the reason included? | The employee's line manager, by email only, through the existing Notification Service (Platform team). The email includes the employee's reason when one was given. In-app notifications are out of scope. | ArunVijay-VS | 2026-11-26 |
| DEC-05 | Unit type and Jira story | Type is API + UI. There is no Jira story yet. | ArunVijay-VS | 2026-11-26 |
| DEC-06 | Which lane does this unit use? | business-change. The SoR does not accept a new-feature lane, so the unit uses business-change, the closest lane it accepts. | ArunVijay-VS | 2026-11-26 |
