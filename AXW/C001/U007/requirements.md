# Employee Leave Request Cancellation – Requirements

<!-- Produced by: ba-unit-requirements | Input: intent.md | Consumed by: architect-unit-design -->

| Field | Details |
|---|---|
| Unit ID | AXW-C001-U007 |
| Capability | AXW-C001 |
| Source Intent | AXW-C001-U007/intent.md (SoR artifact) |
| Status | success |

## 1. Story

**As an** employee, **I want** to cancel my own leave requests from the My Leave page, **so that** I can withdraw leave when my plans change without emailing HR and waiting for them to cancel it manually.

## 2. Scope

**In:**
- An employee can cancel a whole leave request of their own from My Leave:
  - a pending request at any time, whatever its dates;
  - an approved request up to and including the day before its first day of leave, judged by the employee's local date.
- The employee confirms before the cancellation happens and may give a reason of up to 500 characters.
- Cancelling an approved request returns the days that were deducted when it was approved.
- The line manager is emailed. A problem with the email never stops the cancellation.
- The cancelled request stays visible with the status Cancelled.
- The system refuses, and says why:
  - cancelling approved leave on or after its first day (the employee is told to contact HR);
  - cancelling a rejected or already-cancelled request;
  - cancelling someone else's request.

**Out:**
- Changes to how managers approve or reject.
- HR cancelling on an employee's behalf (the back office already does this).
- Cancelling some of the days in a request.
- In-app notifications.

## 3. Functional Requirements

| ID | Requirement |
|---|---|
| FR-01 | The system shall let an employee cancel their own leave request when it is pending, or when it is approved and the employee's local date is before its first day of leave. It shall refuse every other cancellation and tell the employee why. |
| FR-02 | The system shall ask the employee to confirm before cancelling, and shall change nothing if they do not confirm. |
| FR-03 | The system shall accept an optional cancellation reason of up to 500 characters. |
| FR-04 | When an approved request is cancelled, the system shall straight away return to the employee's leave balance the days that were deducted when it was approved. Cancelling a pending request returns nothing, because nothing was deducted. |
| FR-05 | The system shall email the employee's line manager when a request is cancelled, including the reason when one was given, without the cancellation depending on that email being sent. |
| FR-06 | The system shall keep a cancelled request visible on the employee's My Leave page with the status Cancelled and the reason, if one was given. |

## 4. Acceptance Criteria

**Setup**
- Employee 1 (E0001) is the employee cancelling, and today is 9 Dec 2026 on Employee 1's local calendar.

**AC-01 – Whether a request can be cancelled depends on its status and first day (FR-01)**
- Given Employee 1 has a leave request with the status and dates below
- When Employee 1 cancels it from My Leave and confirms
- Then the outcome for each case below

| Case | Status | Dates | Outcome |
|---|---|---|---|
| Pending, leave in the future | Pending | 15–17 Dec 2026 | Cancelled |
| Pending, first day already passed, no decision yet | Pending | 1–3 Dec 2026 | Cancelled |
| Approved, first day is tomorrow (last allowed day) | Approved | 10–12 Dec 2026 | Cancelled |
| Approved, first day is today | Approved | 9–11 Dec 2026 | Refused: told the leave has started and to contact HR to cancel it |
| Approved, leave in progress | Approved | 7–11 Dec 2026 | Refused: told the leave has started and to contact HR to cancel it |
| Rejected | Rejected | 15–17 Dec 2026 | Refused: told this request cannot be cancelled |
| Already cancelled | Cancelled | 15–17 Dec 2026 | Refused: told this request cannot be cancelled |

When a request is cancelled, the whole request is cancelled. When a cancellation is refused, the request's status and Employee 1's leave balance are unchanged and no email is sent.

**AC-02 – An employee cannot cancel someone else's request (FR-01)**
- Given Employee 2 (E0002) has a Pending request for 15–17 Dec 2026
- When Employee 1 tries to cancel it
- Then Employee 1 is told this request cannot be cancelled, the request stays Pending and no email is sent

**AC-03 – The cut-off uses the employee's local date, not the company's (FR-01)**
- Given Employee 1 has an Approved request with the first day below, while the company's date differs from Employee 1's
- When Employee 1 cancels it and confirms
- Then the outcome for each case below

| Case | First day of leave | Company's date | Outcome |
|---|---|---|---|
| Employee on the day before; company already on the first day | 10 Dec 2026 | 10 Dec 2026 | Cancelled |
| Employee on the first day; company still on the day before | 9 Dec 2026 | 8 Dec 2026 | Refused: told the leave has started and to contact HR to cancel it |

**AC-04 – Employee confirms before cancelling (FR-02)**
- Given Employee 1 has an Approved 3-day request for 15–17 Dec 2026, a leave balance of 9 days, and has chosen to cancel it and been asked to confirm cancelling the leave for 15–17 Dec 2026
- When Employee 1 answers the confirmation
- Then the outcome for each case below

| Case | Answer | Outcome |
|---|---|---|
| Confirmed | Yes | Request Cancelled; balance becomes 12 days |
| Backed out | No | Request stays Approved; balance stays 9 days; no email is sent |

**AC-05 – Optional reason of up to 500 characters (FR-03)**
- Given Employee 1 has a Pending request for 15–17 Dec 2026
- When Employee 1 cancels it with the reason below and confirms
- Then the outcome for each case below

| Case | Reason | Outcome |
|---|---|---|
| No reason | (empty) | Cancelled, with no reason |
| At the limit | 500 characters | Cancelled, with the full 500-character reason |
| Over the limit | 501 characters | Refused: told the reason can be at most 500 characters; the request stays Pending |

**AC-06 – Deducted days are returned to the balance (FR-04)**
- Given Employee 1 has a 3-day request for 15–17 Dec 2026 with the status and balance below
- When Employee 1 cancels it and confirms
- Then the outcome for each case below

| Case | Status | Balance before | Outcome |
|---|---|---|---|
| Approved: the 3 deducted days are returned | Approved | 9 days | Balance is 12 days straight away |
| Pending: nothing was deducted | Pending | 12 days | Balance stays 12 days |

**AC-07 – The line manager is emailed (FR-05)**
- Given Manager 1 (manager1@example.com) is Employee 1's line manager, and Employee 1 has a Pending request for 15–17 Dec 2026
- When Employee 1 cancels it with the reason below and confirms
- Then the outcome for each case below

| Case | Reason | Outcome |
|---|---|---|
| With reason | Trip postponed | Manager 1 is emailed that Employee 1's leave for 15–17 Dec 2026 has been cancelled, with the reason "Trip postponed" |
| Without reason | (empty) | Manager 1 is emailed that Employee 1's leave for 15–17 Dec 2026 has been cancelled, with no reason |

**AC-08 – The cancellation completes even when the email can't be sent (FR-05)**
- Given Employee 1 has an Approved 3-day request for 15–17 Dec 2026 and a leave balance of 9 days, under the condition below
- When Employee 1 cancels it and confirms
- Then the outcome for each case below

| Case | Condition | Outcome |
|---|---|---|
| Notification Service unavailable | Manager 1 is Employee 1's line manager; the Notification Service is unavailable | Request Cancelled; balance is 12 days |
| No line manager on record | Employee 1 has no line manager on record | Request Cancelled; balance is 12 days; no email is sent |

In both cases Employee 1 sees the request as Cancelled.

**AC-09 – A cancelled request stays visible (FR-06)**
- Given Employee 1 has cancelled the request for 15–17 Dec 2026 with the reason below
- When Employee 1 opens My Leave
- Then the outcome for each case below

| Case | Reason | Outcome |
|---|---|---|
| With reason | Trip postponed | Request listed as Cancelled, showing the reason "Trip postponed" |
| Without reason | (empty) | Request listed as Cancelled, with no reason shown |

## Business Rules

| ID | Rule | Used In |
|---|---|---|
| BR-03 | An approved request can be cancelled up to and including the day before its first day of leave, judged by the employee's local date. From the first day onwards, the employee must contact HR. | AC-01, AC-03 |
| BR-04 | Cancellation applies to the whole request only. | AC-01, AC-06 |
| BR-07 | The cancellation reason is optional, with at most 500 characters. | AC-05, AC-07, AC-09 |
| BR-08 | Every cancellation emails the line manager, with the reason when one was given. A problem with the email never stops the cancellation. | AC-07, AC-08 |

## Dependencies

| Depends On | Provides | Needed For |
|---|---|---|
| Existing leave request submission and approval process | Leave requests with their owner, dates and status (Pending, Approved, Rejected, Cancelled) | FR-01, FR-06 |
| Existing leave balance | The employee's balance, which approved cancellations return days to | FR-04 |
| Employee–line manager relationship | The employee's line manager and their email address | FR-05 |
| Employee's local date / time zone | The calendar date used for the cut-off | FR-01 |
| Notification Service (Platform team) | Email delivery to the line manager, including retries | FR-05 |

## Clarifications

| ID | Question | Decision and Reason | Affects | Status |
|---|---|---|---|---|
| Q-01 | What happens if the email can't be sent, because the Notification Service is unavailable or the employee has no line manager on record? | The cancellation still completes and the employee sees it as Cancelled. Delivery and retries are the Notification Service's job. With no line manager on record, no email is sent. A notification problem must never stop an employee cancelling their own leave. | FR-05, AC-08 | Answered |
| Q-02 | Whose date decides "the day before the first day of leave"? | The employee's own local date. Leave dates are the employee's calendar days, so the cut-off uses the same calendar. | FR-01, AC-03 | Answered |
| Q-03 | What does the employee see on My Leave after cancelling? | The request stays in the list with the status Cancelled and shows the reason, if one was given. This keeps a visible history. | FR-06, AC-09 | Answered |
| Q-04 | Does the employee confirm before the cancellation happens? | Yes. Choosing No changes nothing. Cancelling an approved request can't be undone (getting the leave back needs a new request and a new approval), so an accidental click is costly. | FR-02, AC-04 | Answered |
| Q-05 | Is there a length limit on the reason? | Yes, a maximum of 500 characters. That's enough to explain, and it keeps the manager's email readable. | FR-03, AC-05 | Answered |
| Q-06 | Can an employee cancel a pending request whose first day of leave has already passed? | Yes. A pending request can be cancelled at any time, whatever its dates. The "contact HR once leave has started" cut-off applies only to approved requests. | FR-01, AC-01 | Answered |
