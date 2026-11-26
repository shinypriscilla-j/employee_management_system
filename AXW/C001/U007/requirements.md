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

**In:** An employee cancels a whole leave request of their own from My Leave. A pending request can be cancelled at any time. An approved request can be cancelled up to and including the day before its first day of leave, judged by the employee's local date. The employee confirms before the cancellation happens and can give a reason of up to 500 characters. Cancelling an approved request returns its days to the balance. The line manager is emailed, and if the email can't be sent the cancellation still goes through. The cancelled request stays visible with the status Cancelled.

**Out:** Changes to how managers approve or reject. HR cancelling on an employee's behalf (the back office already does this). Employees cancelling leave that has already started (HR handles these). Cancelling some of the days in a request. In-app notifications. Cancelling requests that are rejected or already cancelled.

## 3. Functional Requirements

| ID | Requirement |
|---|---|
| FR-01 | The system shall let an employee cancel their own leave request when it is pending, or when it is approved and the employee's local date is before the first day of leave, and shall refuse every other cancellation. |
| FR-02 | The system shall ask the employee to confirm before cancelling, and shall change nothing if they do not confirm. |
| FR-03 | The system shall accept an optional cancellation reason of up to 500 characters. |
| FR-04 | The system shall return all of a request's days to the employee's leave balance straight away when an approved request is cancelled, and shall leave the balance unchanged when a pending request is cancelled. |
| FR-05 | The system shall email the employee's line manager when a request is cancelled, including the reason when one was given, without the cancellation depending on that email being sent. |
| FR-06 | The system shall keep a cancelled request visible on the employee's My Leave page with the status Cancelled and the reason, if one was given. |

## 4. Acceptance Criteria

```gherkin
Background:
  Given Employee 1 (E0001) has line manager Manager 1 (manager1@example.com)
  And Employee 1's leave balance is 12 days, after any approved requests have been deducted
  And Employee 1's local date is 9 Dec 2026
```

```gherkin
AC-01 – Employee cancels an eligible request (FR-01)
  Given Employee 1 has a "<status>" leave request for "<dates>"
  When Employee 1 cancels it from My Leave and confirms
  Then the request's status is Cancelled
  And the whole request is cancelled, covering every day in "<dates>"

  Examples:
    | Case                                               | status   | dates              |
    | Pending, leave in the future                       | Pending  | 15–17 Dec 2026     |
    | Pending, first day already passed, no decision yet | Pending  | 1–3 Dec 2026       |
    | Approved, first day is tomorrow (last allowed day) | Approved | 10–12 Dec 2026     |
    | Approved, leave well in the future                 | Approved | 21–25 Dec 2026     |
```

```gherkin
AC-02 – Cancellation is refused (FR-01)
  Given "<request>"
  When Employee 1 tries to cancel it
  Then Employee 1 is told "<message>"
  And the request's status, Employee 1's leave balance and Manager 1's inbox are unchanged

  Examples:
    | Case                         | request                                                 | message                                        |
    | Approved, first day is today | Employee 1's Approved request for 9–11 Dec 2026         | the leave has started; contact HR to cancel it |
    | Approved, leave in progress  | Employee 1's Approved request for 7–11 Dec 2026         | the leave has started; contact HR to cancel it |
    | Rejected request             | Employee 1's Rejected request for 15–17 Dec 2026        | this request cannot be cancelled               |
    | Already cancelled            | Employee 1's Cancelled request for 15–17 Dec 2026       | this request cannot be cancelled               |
    | Not the owner                | Employee 2 (E0002)'s Pending request for 15–17 Dec 2026 | this request cannot be cancelled               |
```

```gherkin
AC-03 – Cut-off uses the employee's local date (FR-01)
  Given Employee 1 has an Approved request whose first day is 10 Dec 2026
  And Employee 1's local date is "<employee date>" while the company's date is "<company date>"
  When Employee 1 tries to cancel it
  Then the cancellation is "<outcome>"

  Examples:
    | Case                                                                 | employee date | company date | outcome             |
    | Employee still on the day before; company already on the first day  | 9 Dec 2026    | 10 Dec 2026  | completed           |
    | Employee already on the first day; company still on the day before  | 10 Dec 2026   | 9 Dec 2026   | refused, contact HR |
```

```gherkin
AC-04 – Employee confirms before cancelling (FR-02)
  Given Employee 1 has an Approved request for 15–17 Dec 2026
  When Employee 1 chooses to cancel it
  Then Employee 1 is asked to confirm cancelling the leave for 15–17 Dec 2026
  When Employee 1 answers "<answer>"
  Then "<outcome>"

  Examples:
    | Case       | answer | outcome                                                                    |
    | Confirmed  | Yes    | the request is Cancelled and the balance becomes 15 days                   |
    | Backed out | No     | the request stays Approved, the balance stays 12 days and no email is sent |
```

```gherkin
AC-05 – Optional reason up to 500 characters (FR-03)
  Given Employee 1 has a Pending request for 15–17 Dec 2026
  When Employee 1 cancels it with the reason "<reason>" and confirms
  Then "<outcome>"

  Examples:
    | Case           | reason         | outcome                                                                                |
    | No reason      | (empty)        | the request is Cancelled                                                               |
    | Reason given   | Trip postponed | the request is Cancelled with the reason "Trip postponed"                              |
    | At the limit   | 500 characters | the request is Cancelled with the full 500-character reason                            |
    | Over the limit | 501 characters | Employee 1 is told the reason can be at most 500 characters; the request stays Pending |
```

```gherkin
AC-06 – Leave balance after cancelling (FR-04)
  Given Employee 1 has a "<status>" request of "<days>" days for "<dates>"
  And Employee 1's leave balance is "<before>" days
  When Employee 1 cancels it and confirms
  Then Employee 1's leave balance is "<after>" days straight away

  Examples:
    | Case                                | status   | days | dates                    | before | after |
    | Approved – days returned            | Approved | 3    | 15–17 Dec 2026           | 9      | 12    |
    | Approved, whole week – all returned | Approved | 5    | 14–18 Dec 2026 (Mon–Fri) | 7      | 12    |
    | Pending – nothing to return         | Pending  | 3    | 15–17 Dec 2026           | 12     | 12    |
```

```gherkin
AC-07 – Line manager is emailed (FR-05)
  Given Employee 1 has a Pending request for 15–17 Dec 2026
  When Employee 1 cancels it with the reason "<reason>" and confirms
  Then Manager 1 is emailed at manager1@example.com that Employee 1's leave for 15–17 Dec 2026 has been cancelled
  And the email "<reason in email>"

  Examples:
    | Case           | reason         | reason in email                      |
    | With reason    | Trip postponed | includes the reason "Trip postponed" |
    | Without reason | (empty)        | includes no reason                   |
```

```gherkin
AC-08 – Email cannot be sent; cancellation still completes (FR-05)
  Given Employee 1 has an Approved request for 15–17 Dec 2026 and a leave balance of 9 days
  And "<condition>"
  When Employee 1 cancels it and confirms
  Then the request is Cancelled
  And Employee 1's leave balance is 12 days
  And "<email outcome>"

  Examples:
    | Case                             | condition                                | email outcome                            |
    | Notification Service unavailable | the Notification Service is unavailable  | Employee 1 sees the request as Cancelled |
    | No line manager on record        | Employee 1 has no line manager on record | no email is sent                         |
```

```gherkin
AC-09 – Cancelled request stays visible (FR-06)
  Given Employee 1 has cancelled the request for 15–17 Dec 2026 with the reason "<reason>"
  When Employee 1 opens My Leave
  Then the request for 15–17 Dec 2026 is listed with the status Cancelled
  And "<reason shown>"

  Examples:
    | Case           | reason         | reason shown                         |
    | With reason    | Trip postponed | the reason "Trip postponed" is shown |
    | Without reason | (empty)        | no reason is shown                   |
```

## Business Rules

| ID | Rule | Used In |
|---|---|---|
| BR-03 | An approved request can be cancelled up to and including the day before its first day of leave, judged by the employee's local date. From the first day onwards, the employee must contact HR. | AC-01, AC-02, AC-03 |
| BR-04 | Cancellation applies to the whole request only. | AC-01, AC-06 |
| BR-07 | The cancellation reason is optional (at most 500 characters). | AC-05, AC-07, AC-09 |
| BR-08 | Every cancellation emails the line manager, with the reason when one was given. A notification problem never stops the cancellation. | AC-07, AC-08 |

## Dependencies

| Depends On | Provides | Needed For |
|---|---|---|
| Existing leave request submission and approval process | Leave requests with their owner, dates and status (Pending, Approved, Rejected, Cancelled) | FR-01, FR-06 |
| Existing leave balance | The employee's balance, which approved cancellations add back to | FR-04 |
| Existing My Leave page | Where the employee sees their requests and starts a cancellation | FR-01, FR-02, FR-06 |
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
