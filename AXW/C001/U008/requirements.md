# Employee Equipment Booking – Requirements

<!-- Produced by: ba-unit-requirements | Input: intent.md | Consumed by: architect-unit-design -->

| Field | Details |
|---|---|
| Unit ID | AXW-C001-U008 |
| Capability | AXW-C001 |
| Source Intent | AXW-C001-U008/intent.md (SoR artifact) |
| Status | success |

## 1. Story

**As an** employee, **I want** to see which shared equipment items are available and reserve one for a specific date in the Equipment Portal, **so that** I can borrow laptops, projectors and testing devices without contacting IT to check availability and book them manually.

## 2. Scope

**In:**
- An employee picks a date in the Equipment Portal and sees every equipment item marked Available, Already Booked or Unavailable, without seeing who booked it.
- An employee books an Available item for one whole day, either today or a future date, judged by the employee's local date.
- An employee sees their own bookings, both upcoming and past, each with the status Confirmed or Cancelled.
- An employee cancels their own Confirmed booking before its booking date, which makes the item available again for that date.
- An on-screen confirmation is shown when a booking is made and when one is cancelled.
- The system refuses, and says why, when an employee tries to:
  - book an item already booked for that date (including by themselves);
  - book an item IT has marked unavailable;
  - book a past date;
  - cancel a booking on or after its booking date, including a booking made for today (they are told to contact IT);
  - cancel someone else's booking.

**Out:**
- IT viewing all upcoming bookings. This will be a separate unit.
- Changes to the equipment inventory or maintenance process.
- Bookings covering more than one day, and bookings for a time slot within a day.
- Email or any other notification outside the portal.
- IT cancelling or managing bookings on an employee's behalf.
- Booking by equipment type or from a pool.
- Any change to existing bookings when IT marks an item unavailable.

## 3. Functional Requirements

| ID | Requirement |
|---|---|
| FR-01 | The system shall show the employee every equipment item for a chosen date, each marked Available, Already Booked or Unavailable, without showing who booked it. |
| FR-02 | The system shall let an employee book an Available item for one whole day on today's date or a future date, judged by the employee's local date. It shall refuse a booking for an item already booked for that date, an item IT has marked unavailable, or a past date, and tell the employee why. |
| FR-03 | The system shall show the employee their own bookings, both upcoming and past, each with the status Confirmed or Cancelled. |
| FR-04 | The system shall let an employee cancel their own Confirmed booking when their local date is before the booking date, which makes the item available again for that date. It shall refuse every other cancellation and tell the employee why. |
| FR-05 | The system shall show the employee a confirmation on screen when they make a booking and when they cancel one. |

## 4. Acceptance Criteria

**Setup**
- Employee 1 (E0001) is using the Equipment Portal, and today is 9 Dec 2026 on Employee 1's local calendar.

**AC-01 – Every item is shown with its status for the chosen date (FR-01)**
- Given, for 15 Dec 2026: Laptop L-01 has no booking; Projector P-07 is booked by Employee 2 (E0002); IT has marked Testing Device T-03 unavailable
- When Employee 1 views equipment for 15 Dec 2026
- Then the outcome for each case below

| Case | Item | Outcome |
|---|---|---|
| Free | Laptop L-01 | Shown as Available |
| Booked by someone else | Projector P-07 | Shown as Already Booked |
| Marked unavailable by IT | Testing Device T-03 | Shown as Unavailable |

No item shows who booked it.

**AC-02 – Booking an item (FR-02)**
- Given the item and situation below; Employee 1 already has a Confirmed booking for Laptop L-02 on 15 Dec 2026; Employee 2 has a Confirmed booking for Projector P-07 on 15 Dec 2026; IT has marked Testing Device T-03 unavailable
- When Employee 1 books the item for the date below
- Then the outcome for each case below

| Case | Item | Date | Outcome |
|---|---|---|---|
| Available, future date | Laptop L-01 | 15 Dec 2026 | Booked: a Confirmed booking for Laptop L-01 for the whole of 15 Dec 2026 |
| Available, today (boundary) | Laptop L-01 | 9 Dec 2026 | Booked: a Confirmed booking for Laptop L-01 for the whole of 9 Dec 2026 |
| Different item, same date as their own booking | Laptop L-01 | 15 Dec 2026 | Booked; Employee 1 now holds both Laptop L-01 and Laptop L-02 for 15 Dec 2026 |
| Past date | Laptop L-01 | 8 Dec 2026 | Refused: told past dates cannot be booked |
| Already booked by someone else | Projector P-07 | 15 Dec 2026 | Refused: told the item is already booked for that date |
| Already booked by themselves | Laptop L-02 | 15 Dec 2026 | Refused: told the item is already booked for that date |
| Marked unavailable by IT | Testing Device T-03 | 15 Dec 2026 | Refused: told the item is unavailable |

When a booking is refused, no booking is created and every existing booking is unchanged.

**AC-03 – Booking dates use the employee's local date, not the company's (FR-02)**
- Given Laptop L-01 is Available, and the company's date differs from Employee 1's as shown below
- When Employee 1 books Laptop L-01 for the date below
- Then the outcome for each case below

| Case | Date booked | Company's date | Outcome |
|---|---|---|---|
| Today for the employee; company already on the next day | 9 Dec 2026 | 10 Dec 2026 | Booked |
| Past for the employee; company still on that day | 8 Dec 2026 | 8 Dec 2026 | Refused: told past dates cannot be booked |

**AC-04 – An employee sees their own bookings (FR-03)**
- Given Employee 1 has these bookings: Laptop L-01 on 15 Dec 2026, Confirmed; Projector P-07 on 2 Dec 2026, Confirmed; Laptop L-02 on 20 Dec 2026, Cancelled; Testing Device T-03 on 18 Dec 2026, Confirmed, with IT marking T-03 unavailable after it was booked. Employee 2 has Laptop L-01 on 16 Dec 2026, Confirmed.
- When Employee 1 opens My Bookings in the Equipment Portal
- Then the outcome for each case below

| Case | Booking | Outcome |
|---|---|---|
| Upcoming | Laptop L-01, 15 Dec 2026 | Listed as Confirmed |
| Past | Projector P-07, 2 Dec 2026 | Listed as Confirmed |
| Cancelled | Laptop L-02, 20 Dec 2026 | Listed as Cancelled |
| Item marked unavailable after booking | Testing Device T-03, 18 Dec 2026 | Listed as Confirmed, unchanged |
| Someone else's booking | Employee 2's Laptop L-01, 16 Dec 2026 | Not listed |

**AC-05 – Cancelling a booking (FR-04)**
- Given the booking below
- When Employee 1 cancels it
- Then the outcome for each case below

| Case | Booking | Outcome |
|---|---|---|
| Own, future date | Employee 1's Laptop L-01, 15 Dec 2026, Confirmed | Cancelled; Laptop L-01 is Available again for 15 Dec 2026 |
| Own, booking date is tomorrow (last day to cancel) | Employee 1's Laptop L-01, 10 Dec 2026, Confirmed | Cancelled; Laptop L-01 is Available again for 10 Dec 2026 |
| Own, booking date is today, including a booking made today | Employee 1's Laptop L-01, 9 Dec 2026, Confirmed | Refused: told the booking date has started and to contact IT |
| Own, booking date has passed | Employee 1's Projector P-07, 2 Dec 2026, Confirmed | Refused: told the booking date has started and to contact IT |
| Someone else's booking | Employee 2's Laptop L-01, 16 Dec 2026, Confirmed | Refused: told this booking cannot be cancelled |

When a cancellation is refused, the booking stays Confirmed and the item's status for that date is unchanged.

**AC-06 – Cancellation dates use the employee's local date, not the company's (FR-04)**
- Given Employee 1 has a Confirmed booking for Laptop L-01 on the date below, and the company's date differs from Employee 1's as shown
- When Employee 1 cancels it
- Then the outcome for each case below

| Case | Booking date | Company's date | Outcome |
|---|---|---|---|
| Employee on the day before; company already on the booking date | 10 Dec 2026 | 10 Dec 2026 | Cancelled |
| Employee on the booking date; company still on the day before | 9 Dec 2026 | 8 Dec 2026 | Refused: told the booking date has started and to contact IT |

**AC-07 – An on-screen confirmation is shown (FR-05)**
- Given Laptop L-01 is Available for 15 Dec 2026
- When Employee 1 completes the action below
- Then the outcome for each case below

| Case | Action | Outcome |
|---|---|---|
| Booking made | Books Laptop L-01 for 15 Dec 2026 | Shown a confirmation that Laptop L-01 is booked for 15 Dec 2026 |
| Booking cancelled | Cancels their booking for Laptop L-01 on 15 Dec 2026 | Shown a confirmation that the booking for Laptop L-01 on 15 Dec 2026 is cancelled |

## Business Rules

| ID | Rule | Used In |
|---|---|---|
| BR-01 | Each equipment item can be booked only once per date, whoever books it. | AC-01, AC-02 |
| BR-03 | An item IT has marked unavailable cannot be booked. | AC-01, AC-02 |
| BR-06 | A booking can be made for today or any future date, judged by the employee's local date. Past dates cannot be booked. | AC-02, AC-03 |
| BR-07 | An employee can cancel only their own booking, and only before the booking date, judged by the employee's local date. From the booking date onwards, cancellation is handled by IT. | AC-05, AC-06 |

## Dependencies

| Depends On | Provides | Needed For |
|---|---|---|
| Existing equipment inventory | The specific equipment items that can be booked | FR-01, FR-02 |
| Existing IT-managed availability status | Which items IT has marked unavailable | FR-01, FR-02 |
| Employee's local date / time zone | The calendar date used for booking and cancellation cut-offs | FR-02, FR-04 |

## Clarifications

| ID | Question | Decision and Reason | Affects | Status |
|---|---|---|---|---|
| Q-01 | Is it the Equipment Portal or the Employee Portal? | The Equipment Portal, used consistently across the story, requirements, acceptance criteria and UI references. | Story, AC-04, Setup | Answered |
| Q-02 | Can an employee cancel a booking they made for today? | No. Employees can cancel only before the booking date, so a booking made for today must be cancelled by IT. | FR-04, AC-05 | Answered |
| Q-03 | Whose date decides "today", "a past date" and "before the booking date"? | The employee's local date, consistent with the leave-cancellation unit. All booking date checks use it. | FR-02, FR-04, AC-03, AC-06 | Answered |
| Q-04 | When an employee picks a date, what do they see? | All equipment for that date, each clearly marked Available, Already Booked or Unavailable. Employees do not see who booked an item. | FR-01, AC-01 | Answered |
| Q-05 | What does My Bookings include? | The employee's upcoming and past bookings. Cancelled bookings stay visible with the status Cancelled. A booking whose item IT has since marked unavailable stays valid, unchanged, and keeps the status Confirmed. | FR-03, AC-04 | Answered |
