# Design — Even or Odd Number Checker

**Unit:** AXW-C001-U006 · **Kind:** UI · **Status:** Approved by User (BA), 2026-11-25

## Screen
One page in the browser. No API, nothing is stored.
Layout: [screen.xml](./screen.xml)

| Component | What it does | R-IDs |
|---|---|---|
| Number field | Where the user types the number | R1, R3–R6 |
| Check button | Checks the number | R1, R4–R6 |
| Clear button | Empties the field and the message | R2 |
| Message | Shows the result or the error | R1, R4–R6 |

## How it works
Check judges the input in this order, ignoring spaces around it:
1. Empty → `input_empty`
2. Not a whole number (an optional "+" or "-" is allowed) → `input_not_integer`
3. More than 10 digits, leading zeros included → `input_too_long`
4. Otherwise the last digit decides: 0, 2, 4, 6 or 8 → "Even Number", anything else → "Odd Number"

On an error, the field is marked invalid and keeps the text.
Editing the field removes the message. Clear resets the screen.

## Reason codes

| Code | Message | R-ID |
|---|---|---|
| `input_empty` | Please enter a number | R4 |
| `input_not_integer` | Please enter a valid integer | R5 |
| `input_too_long` | Please enter a number with up to 10 digits | R6 |

## Decisions
- Pressing Enter also runs Check (requirements A1 doesn't require it, but doesn't forbid it).
- Typing isn't capped at 10 digits, so the too-long message can appear.
- Built separately from AXW-C001-U005; nothing is shared.
- Requirements assumptions A1–A3 approved by User (BA), 2026-11-25.

## Coverage

| R-ID | Where |
|---|---|
| R1 | How it works, step 4 |
| R2 | How it works, Clear |
| R3 | How it works, editing the field |
| R4–R6 | How it works, steps 1–3 · Reason codes |
