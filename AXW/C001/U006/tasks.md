# Tasks — Even or Odd Number Checker

**Unit:** AXW-C001-U006 · **Surface:** UI · **Built from:** requirements.md (df67200) · design.md (f820700) · screen.xml (425353b)
**Status:** Ready · **Approved by:** User (BA), 2026-11-25

## Build order
- [ ] Build the screen as laid out in [screen.xml](./screen.xml): heading, Number field, Check and Clear buttons, message area — R1–R6 — *done: the page opens empty and matches screen.xml*
- [ ] Judge the input as in design "How it works", steps 1–4, producing `input_empty`, `input_not_integer` (assumes A3), `input_too_long` (assumes A2) or Even/Odd — R1, R4–R6 — *done: each outcome is returned for its kind of input*
- [ ] Check (and Enter, assumes A1) shows the result or the reason code's message; on an error the field is marked invalid and keeps the text — R1, R4–R6 — *done: every message from design "Reason codes" can be shown*
- [ ] Editing the field removes the shown message — R3 — *done: no message after any edit until Check*
- [ ] Clear empties the field and removes the message — R2 — *done: the screen is back to first use*
- [ ] Acceptance tests for R1–R6, using the examples in each requirement — R1–R6 — *done: all pass*

## Coverage

| R-ID | Build | Test |
|---|---|---|
| R1 | Judge the input · Check | Acceptance tests |
| R2 | Clear | Acceptance tests |
| R3 | Editing the field | Acceptance tests |
| R4 (`input_empty`) | Judge the input · Check | Acceptance tests |
| R5 (`input_not_integer`) | Judge the input · Check | Acceptance tests |
| R6 (`input_too_long`) | Judge the input · Check | Acceptance tests |

The Number field rule is built by "Judge the input"; "Any user, not stored" needs no task.

## Blocked and open
None. Assumptions A1–A3 were approved by User (BA), 2026-11-25.
