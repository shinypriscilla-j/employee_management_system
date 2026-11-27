# Intent – Employee Equipment Booking

| Field | Details |
|---|---|
| **Unit** | `AXW-C001-U008 – equipment-booking` |
| **Capability** | `AXW-C001` |
| **Jira Story** | None yet |
| **Lane** | `business-change` |
| **Type** | `API + UI` |

## User Story

**As an** employee, **I want** to see which shared equipment items are available and reserve one for a specific date in the Equipment Portal, **so that** I can borrow laptops, projectors and testing devices without contacting IT to check availability and book them manually.

## Scope

| ID | In Scope |
|---|---|
| IN-01 | An employee can see which equipment items are available to book for a chosen date. |
| IN-02 | An employee can book a specific available equipment item for one whole day, either today or a future date. |
| IN-03 | An employee can see their own existing bookings in the Equipment Portal. |
| IN-04 | An employee can cancel their own booking up to the day before the booking date. |
| IN-05 | The employee sees a confirmation on screen in the portal when a booking is made and when one is cancelled. |
| IN-06 | The employee is prevented from booking an item that is already booked for that date, an item IT has marked unavailable, or a past date. |
| IN-07 | The employee is prevented from cancelling a booking on or after the booking date. That cancellation goes to IT. |

| ID | Out of Scope |
|---|---|
| OUT-01 | IT viewing all upcoming bookings. This will be a separate unit. |
| OUT-02 | Changes to the equipment inventory or maintenance process. |
| OUT-03 | Bookings covering more than one day, and bookings for a time slot within a day. |
| OUT-04 | Email or any other notification outside the portal. |
| OUT-05 | IT cancelling or managing bookings on an employee's behalf. |
| OUT-06 | Booking by equipment type or from a pool, such as "any projector". Bookings are for one specific item. |
| OUT-07 | Any change to existing bookings when IT marks an item unavailable. |

## Business Rules

| ID | Rule | Example |
|---|---|---|
| BR-01 | Each equipment item can be booked only once per date, whoever books it. | Employee 1 books Projector P-07 for 15 Dec. Employee 2 cannot book Projector P-07 for 15 Dec, but can book it for 16 Dec. |
| BR-02 | An employee cannot book the same item twice for the same date, but can book different items for the same date. | Employee 1 has Laptop L-01 for 15 Dec. They cannot book Laptop L-01 again for 15 Dec, but can book Projector P-07 for 15 Dec. |
| BR-03 | An item IT has marked unavailable cannot be booked. | IT marks Laptop L-01 unavailable. No employee can book Laptop L-01 for any date while it is marked unavailable. |
| BR-04 | Marking an item unavailable blocks only new bookings. Existing bookings for that item stay as they are. | Employee 1 has Laptop L-01 for 15 Dec. IT then marks Laptop L-01 unavailable. Employee 1's booking for 15 Dec stays. |
| BR-05 | A booking covers one whole day. | Employee 1 books Projector P-07 for 15 Dec. The booking is for all of 15 Dec. They cannot book it for 15–16 Dec or for 9:00–12:00 only. |
| BR-06 | A booking can be made for today, as long as it is made before today ends, or for any future date. Past dates cannot be booked. | On 9 Dec, Employee 1 can book for 9 Dec or 10 Dec, but not for 8 Dec. |
| BR-07 | An employee can cancel only their own booking, and only before the booking date. From the booking date onwards, cancellation is handled by IT. | Employee 1 has a booking for 15 Dec. Cancelling on 14 Dec is allowed. Cancelling on 15 Dec is not, and Employee 1 must contact IT. |
| BR-08 | The employee is shown a confirmation in the portal whenever they make or cancel a booking. | Employee 1 books Projector P-07 for 15 Dec and sees a confirmation of that booking on screen. |

## Dependencies

| Depends On | Why It Is Needed |
|---|---|
| Existing Equipment Portal | This is where the employee views equipment, books it and sees their bookings. |
| Existing equipment inventory | Provides the specific equipment items that can be booked, such as Laptop L-01 and Projector P-07. |
| Existing IT-managed availability status | Shows which items IT has marked unavailable, so they cannot be booked. |

## Decisions

| ID | Question | Decision | By | Date |
|---|---|---|---|---|
| DEC-01 | Is a booking for a specific item or for a type of equipment? | A specific physical item. Each item can be booked only once per date, by anyone. An employee can book different items for the same date but not the same item twice. | ArunVijay-VS | 2026-11-27 |
| DEC-02 | What does a booking cover, and can it be for today? | One whole day only. Multi-day bookings and time slots are out of scope. Today can be booked before the day ends; past dates cannot. | ArunVijay-VS | 2026-11-27 |
| DEC-03 | Is IT's view of all upcoming bookings part of this unit, and what already exists? | The IT view will be a separate unit. The portal, the equipment inventory and IT's availability status already exist. This unit covers only employees booking equipment and their own bookings. | ArunVijay-VS | 2026-11-27 |
| DEC-04 | How is the employee given confirmation? | On screen in the portal only. No email or other notification. | ArunVijay-VS | 2026-11-27 |
| DEC-05 | What happens to existing bookings when IT marks an item unavailable? | They stay as they are. Being unavailable blocks new bookings only. | ArunVijay-VS | 2026-11-27 |
| DEC-06 | Unit type, lane and Jira story | Type is API + UI. The lane is business-change, because the SoR does not accept new-feature. There is no Jira story yet. | ArunVijay-VS | 2026-11-27 |
