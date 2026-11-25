# Requirements — Even or Odd Number Checker

| | |
|---|---|
| **Unit** | AXW-C001-U006 |
| **Parent feature** | AXW-C001 — Features |
| **Surface** | UI |
| **Status** | Ready |
| **Approved by** | User (BA), 2026-11-25 |

Non-functional and UX behaviour follow the project quality baseline unless §5 says otherwise.

## 1. Story
As a **user**, I want to **enter a number and check whether it is even or odd**, so that **I can easily identify the number type**.

## 2. Field rules

| Field | Rule | Who may change it |
|---|---|---|
| Number | A whole number of up to 10 digits, with an optional "+" or "-" sign. Spaces around it are ignored. | Any user. Not stored. |

## 3. Acceptance criteria

### Happy path
| R-ID | Given | When | Then |
|---|---|---|---|
| R1 | A valid number is entered, e.g. 12, 7, 0, -3, +7, " 12 ", -9999999999 | The user clicks Check | "Even Number" if it divides by 2, otherwise "Odd Number" |
| R2 | A number and its result are shown | The user clicks Clear | The input and the result are both cleared |
| R3 | A result is shown | The user edits the number | The result disappears until Check is clicked again |

### Failure cases
| R-ID | Class | Given | When | Then |
|---|---|---|---|---|
| R4 | input | Nothing, or only spaces, is entered | The user clicks Check | "Please enter a number" (`input_empty`) |
| R5 | input | The input isn't a whole number, e.g. "abc", "3.5" | The user clicks Check | "Please enter a valid integer" (`input_not_integer`); the input is kept |
| R6 | input | The number has more than 10 digits, e.g. 12345678901 | The user clicks Check | "Please enter a number with up to 10 digits" (`input_too_long`); the input is kept |

**Not applicable:** authorization (no sign-in), repetition (clicking Check again gives the same answer), concurrency, dependency, partial failure, ordering, time, state (a single screen with nothing stored and no other systems).
**Not decided:** None.

## 4. Reason codes

| Code | Meaning | R-ID |
|---|---|---|
| `input_empty` | Nothing was entered | R4 |
| `input_not_integer` | The input isn't a whole number | R5 |
| `input_too_long` | The number has more than 10 digits | R6 |

## 5. Differences from the project baseline
None.
No project baseline exists yet, so these stay open for this unit: accessibility, supported browsers and devices, response speed.

## 6. Open questions and assumptions

| # | Item | Blocking? | Owner | Answer |
|---|---|---|---|---|
| Q1 | Spaces, "+" sign, maximum length, message when too long, result on edit | No | User (BA) | As in §2 and R1–R6. User (BA), 2026-11-25 |
| A1 | Only the Check button triggers a check; Enter isn't required (R1) | — | User (BA) | |
| A2 | Leading zeros count toward the 10 digits (R6) | — | User (BA) | |
| A3 | Input that isn't a whole number gets `input_not_integer`, even if it is also too long (R5, R6) | — | User (BA) | |

**Out of this story:** storing input or results; any server or API part; changes to AXW-C001-U005.
