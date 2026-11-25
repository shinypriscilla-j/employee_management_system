# Tasks — Even or Odd Number Checker

| | |
|---|---|
| **Unit** | AXW-C001-U005 |
| **Surface** | UI |
| **Built from** | requirements.md (c42728e) · design.md (b746a3f) · contracts/ (none — design § Contracts lists no files) |
| **Status** | Ready |
| **Approved by** | Story requester, 2026-11-25 |

> Build order. One task ≈ one commit / PR. Each task names the files and classes it touches (from design
> § Implementation), its R-IDs and how we know it's done.
> All paths are relative to the root of the existing project UI repo (design D8).

## 1. Contracts & data
N/A — because the design lists no contract files and the unit owns no data (design § API, § Data, § Contracts).

## 2. Field rules & authorization
- [ ] Define the outcome and parse types per design § Implementation: `CheckOutcome` (`'even' | 'odd' | 'number_missing' | 'not_an_integer'`) and `ParseResult` in `src/features/even-odd-checker/types.ts` — R2–R23 — *done: the types compile; nothing else is exported*
- [ ] Validate Number per the Number field rule, design "Number checker rule" steps 1–3: `parseInteger(raw)` in `src/features/even-odd-checker/logic/checkNumber.ts` removes leading and trailing U+0020 spaces only (D7); empty → `{ kind: 'empty' }`; matches `^[+-]?[0-9]+$` → `{ kind: 'integer', digits }` with the sign removed; anything else → `{ kind: 'invalid' }`. No length limit — R7, R8, R9, R10, R16, R17, R18, R19, R20, R21, R22, R23 — *done: unit tests for these R-IDs pass*
- [ ] Apply the Even/Odd rule per design D2: `classifyParity(digits)` in `src/features/even-odd-checker/logic/checkNumber.ts` returns `even` when the last digit is 0, 2, 4, 6 or 8, otherwise `odd`; never converts to `Number` or `BigInt` — R2, R3, R4, R5, R6, R7, R8, R9, R10, R11, R12 — *done: unit tests pass, including a 1,000-digit value*
- [ ] Combine them per design C3: `checkNumber(raw)` in `src/features/even-odd-checker/logic/checkNumber.ts` maps `empty` → `number_missing`, `invalid` → `not_an_integer`, `integer` → `classifyParity(digits)`; total over any string, never throws — R2–R12, R16–R23 — *done: unit tests pass for every R-ID example value and the edge values*
- [ ] Who may change it: anyone, with no login; takes effect only on Check or Enter; nothing saved. `EvenOddChecker` keeps `value` and `outcome` in component state only, with no local storage, session storage, cookies or URL writes (design § Data, § Cross-cutting) — R1, R13 — *done: the e2e test confirms storage is empty after a check and a reload starts empty*

## 3. Backend behaviour
N/A — because the unit has no API or server (design § API). The checker logic is client-side and built in §2.

## 4. UI
- [ ] Add `@mui/icons-material` 6.1 as a dependency of the UI repo, per design D4 and § Implementation Stack — R16–R23 — *done: `ErrorOutline` imports and the build passes*
- [ ] Copy constants per design § Validation & copy: `messages` in `src/features/even-odd-checker/messages.ts` with keys `title`, `numberLabel`, `check`, `clear`, `even`, `odd`, `number_missing`, `not_an_integer`, holding exactly the English strings in that table — R2–R23 — *done: the strings match the acceptance criteria character for character*
- [ ] Standalone page entry per design C7 and D5:
  - `even-odd-checker.html`, with `<title>` from `messages.title`, `<div id="root">` and the module script
  - the `evenOddChecker` input in `build.rollupOptions.input` in `vite.config.ts`
  - `src/features/even-odd-checker/main.tsx`, which renders `<ThemeProvider><CssBaseline/><EvenOddChecker/></ThemeProvider>`

  — R1 — *done: `/even-odd-checker.html` opens in dev and in the build output; no other page links to it*
- [ ] Result message per design C5: `ResultMessage` in `src/features/even-odd-checker/components/ResultMessage.tsx`, inputs `outcome: CheckOutcome | null` and `id`.
  - It's an always-present `role="status"` region that renders `messages[outcome]`, or nothing when `outcome` is null.
  - It sets `data-outcome` to the outcome, or no attribute when null.
  - For `number_missing` and `not_an_integer` it uses the error colour, preceded by `ErrorOutline` with `aria-hidden`.

  — R2–R12, R16–R23 — *done: component test renders all five states*
- [ ] Checker screen — empty (first use) state, per design § Screens layout and § Components: `EvenOddChecker` in `src/features/even-odd-checker/EvenOddChecker.tsx`.
  - Structure: `<main>` → `Container` (max width 480 px) → `Paper` → `<form noValidate>` labelled by the `<h1>` (`messages.title`).
  - The Number `TextField`: `type="text"`, no `inputMode`, `autoComplete="off"`, label `messages.numberLabel`, no `required`.
  - Buttons: Check (`variant="contained"`, `type="submit"`) and Clear (`variant="outlined"`, `type="button"`).
  - `ResultMessage` with `outcome` null.

  — R1 — *done: first load shows an empty field, both buttons and an empty status region*
- [ ] Checker screen — populated state: `handleChange` stores the typed text as entered; no result is shown until Check — R1 — *done: typing "42" shows "42" and no result*
- [ ] Checker screen — success state: `handleSubmit` (the form's submit, from a Check click or Enter) calls `preventDefault`, sets `outcome = checkNumber(value)` and leaves `value` unchanged; "Even Number" or "Odd Number" shows in the status region — R2, R3, R4, R5, R6, R7, R8, R9, R10, R11, R12, R13 — *done: the R2–R13 e2e tests pass*
- [ ] Checker screen — error (recoverable) state, per design § Reason code → UI:
  - For `number_missing` and `not_an_integer`, the `TextField` gets `error`, `aria-invalid="true"` and `aria-describedby` pointing at the `ResultMessage` id.
  - The typed text is kept, and no Even/Odd result is shown.

  — R16, R17, R18, R19, R20, R21, R22, R23 — *done: the R16–R23 e2e tests pass, and the field still holds "4.5", "4.0", "abc", "1e3", "1,000" or "-" as typed*
- [ ] Checker screen — cleared state: `handleClear` sets `value` to `""` and `outcome` to `null`; with nothing typed and nothing shown, the page stays as it was — R14, R25 — *done: the R14 and R25 e2e tests pass*
- [ ] Checker screen — edited after a result (design F3): `handleChange` sets `outcome` to `null` in the same update as the new value, so a shown result or message disappears straight away and nothing shows until the next Check — R15 — *done: the R15 e2e test passes for a result and for a message*
- [ ] Repetition per design "State & idempotency": `outcome` is a single value, so repeated submits, a double-click, or Enter followed by a click show exactly one message; nothing is appended — R24 — *done: the R24 component and e2e tests show one message after a double-click*
- [ ] Checker screen — offline on submit: no network call is made anywhere on the page, so Check works with the browser offline (design § States) — R2–R23 — *done: the e2e offline test passes*
- [ ] Validation timing per design § Validation & copy: `checkNumber` runs only in `handleSubmit`, never in `handleChange` (no live result) — R15 — *done: component test shows no result while typing*
- [ ] Keyboard map and focus rules per design § Accessibility:
  - Tab order is Number → Check → Clear, and Enter in the field submits.
  - The field has `autoFocus` on entry.
  - On `even` / `odd`, focus stays where it was.
  - On `number_missing` / `not_an_integer`, focus moves to the Number field.
  - After Clear, focus moves to the Number field (via a ref on the input).

  — R13, R14, R16–R23, R25 — *done: component tests assert `document.activeElement` after each outcome*
- [ ] Test IDs per design § Test IDs: `even-odd-page`, `even-odd-title`, `even-odd-input`, `even-odd-check-button`, `even-odd-clear-button` and `even-odd-result` on the elements that table names — R1–R25 — *done: every e2e selector uses these IDs only*

## 5. Non-functional (differences from the project baseline)
- [ ] Meet accessibility (requirements §5; design § Accessibility):
  - The whole page works with the keyboard alone.
  - Each result or message is announced through the `role="status"` region.
  - MUI default-theme contrast is used, the layout holds at 320 px with no horizontal scroll and at 200% zoom, and errors carry text plus an icon, not colour alone.

  Built by the §4 tasks above; this task checks the layout at 320 px and 200% zoom by hand against design § Screens — requirements §5 — *done: the verify task below passes*
- [ ] Verify accessibility: `e2e/even-odd-checker.a11y.spec.ts` (`@axe-core/playwright`) reports no WCAG 2.2 AA violations in the initial, success ("Even Number") and error ("Please enter a valid integer") states, and finds no horizontal scroll at a 320 px viewport — requirements §5 — *done: the spec passes*
- [ ] Meet browsers (requirements §5; design § Cross-cutting Browsers): the page uses only the stack's React and MUI components and adds no browser-specific code — requirements §5 — *done: the verify task below passes*
- [ ] Verify browsers: `e2e/even-odd-checker.spec.ts` runs on Chromium, Firefox and WebKit, and on mobile Chrome and mobile Safari using Playwright's built-in `devices` profiles — requirements §5 — *done: every R-ID test passes on all five*

## 6. Errors, logging, metrics
- [ ] Every reason code is produced and asserted (see Reason-code coverage) — R16–R23 — *done: both codes are produced by `checkNumber` and asserted through `data-outcome` in unit and e2e tests*
- [ ] Logs, metrics and audit: N/A — because design § Cross-cutting names none (no server; no analytics or telemetry is added)

## 7. Tests
- [ ] Unit tests in `src/features/even-odd-checker/logic/checkNumber.test.ts` (Vitest), table-driven:
  - `"42"`→even, `"7"`→odd, `"0"`→even, `"-4"`→even, `"-3"`→odd, `" 42 "`→even, `"+7"`→odd, `"007"`→odd, `"-0"`→even
  - `"98765432109876543211"`→odd, `"98765432109876543210"`→even
  - `""` and `"   "`→`number_missing`
  - `"4.5"`, `"4.0"`, `"abc"`, `"1e3"`, `"1,000"`, `"-"`→`not_an_integer`
  - edge values: `"+0"`→even, `"000"`→even, `"\t42"`→`not_an_integer`, `"٤"`→`not_an_integer`, a 1,000-digit number ending in 7→odd

  — R2–R12, R16–R23 — *done: all pass*
- [ ] Component tests in `src/features/even-odd-checker/EvenOddChecker.test.tsx` (Vitest + React Testing Library + user-event):
  - Enter submits (R13); Clear empties both (R14); editing removes the message (R15); a double-click shows one message (R24); Clear on an empty page changes nothing (R25)
  - no result while typing
  - focus after success, after an error and after Clear
  - `aria-invalid` on error, and `data-outcome` values

  — R13, R14, R15, R24, R25 — *done: all pass*
- [ ] Acceptance tests in `e2e/even-odd-checker.spec.ts` (Playwright), one `test('R<n> …')` per R-ID using its Given / When / Then values exactly, selecting by `data-testid` only (see Coverage). The file also covers:
  - offline on submit: `context.setOffline(true)`, then "42" + Check → "Even Number"
  - nothing stored: storage is empty after a check, and a reload shows an empty field and no result

  — R1–R25 — *done: all pass on the five browser profiles*
- [ ] Must-NOT-offer check per design § Must NOT offer, in `e2e/even-odd-checker.spec.ts`:
  - no history list, no second number field, no other check, no links or navigation
  - the input is `type="text"` with no `inputMode`
  - Check is never disabled
  - no browser validation bubble on an empty submit

  — R1, R5, R6, R8, R15, R16 — *done: assertions pass*

## Coverage

| R-ID | Kind | Build task(s) | Test task(s) |
|---|---|---|---|
| R1 | happy path | UI: Standalone page entry; empty (first use) state; populated state | Tests: acceptance R1 |
| R2 | happy path | Field rules: Even/Odd rule; combine. UI: success state | Tests: unit; acceptance R2 |
| R3 | happy path | Field rules: Even/Odd rule; combine. UI: success state | Tests: unit; acceptance R3 |
| R4 | happy path | Field rules: Even/Odd rule. UI: success state | Tests: unit; acceptance R4 |
| R5 | happy path | Field rules: validate Number; Even/Odd rule. UI: success state | Tests: unit; acceptance R5 |
| R6 | happy path | Field rules: validate Number; Even/Odd rule. UI: success state | Tests: unit; acceptance R6 |
| R7 | happy path | Field rules: validate Number (spaces). UI: success state | Tests: unit; acceptance R7 |
| R8 | happy path | Field rules: validate Number ("+"). UI: success state | Tests: unit; acceptance R8 |
| R9 | happy path | Field rules: validate Number (leading zeros). UI: success state | Tests: unit; acceptance R9 |
| R10 | happy path | Field rules: validate Number ("-0"). UI: success state | Tests: unit; acceptance R10 |
| R11 | happy path | Field rules: Even/Odd rule (last digit). UI: success state | Tests: unit; acceptance R11 |
| R12 | happy path | Field rules: Even/Odd rule (last digit). UI: success state | Tests: unit; acceptance R12 |
| R13 | happy path | UI: success state (form submit); keyboard map and focus | Tests: component; acceptance R13 |
| R14 | happy path | UI: cleared state; focus | Tests: component; acceptance R14 |
| R15 | happy path | UI: edited after a result; validation timing | Tests: component; acceptance R15 |
| R16 | failure — input | Field rules: validate Number (empty). UI: error state; reason code → UI | Tests: unit; acceptance R16 |
| R17 | failure — input | Field rules: validate Number (only spaces). UI: error state | Tests: unit; acceptance R17 |
| R18 | failure — input | Field rules: validate Number. UI: error state | Tests: unit; acceptance R18 |
| R19 | failure — input | Field rules: validate Number. UI: error state | Tests: unit; acceptance R19 |
| R20 | failure — input | Field rules: validate Number. UI: error state | Tests: unit; acceptance R20 |
| R21 | failure — input | Field rules: validate Number. UI: error state | Tests: unit; acceptance R21 |
| R22 | failure — input | Field rules: validate Number. UI: error state | Tests: unit; acceptance R22 |
| R23 | failure — input | Field rules: validate Number. UI: error state | Tests: unit; acceptance R23 |
| R24 | failure — repetition | UI: repetition per "State & idempotency" | Tests: component; acceptance R24 |
| R25 | failure — state | UI: cleared state | Tests: component; acceptance R25 |

**Not applicable (from requirements):** authorization (no login; anyone may use the page), concurrency (one person on their own page), dependency (no other system), partial failure (nothing is saved), ordering (no events), time (no time-based rules).
**Design state rows with no task (N/A in design § States):** loading, empty — filtered, partial, submitting, error — terminal, disabled / unauthorized, offline — on load, session expiry.

## Reason-code coverage

| Code | R-ID | Produced by | Asserted by |
|---|---|---|---|
| `number_missing` | R16, R17 | Field rules: combine (`checkNumber`); UI: error state | Tests: unit; acceptance R16, R17 |
| `not_an_integer` | R18, R19, R20, R21, R22, R23 | Field rules: combine (`checkNumber`); UI: error state | Tests: unit; acceptance R18–R23 |

## Field-rule coverage

| Field | Rule task | Who-may-change task |
|---|---|---|
| Number | Field rules: validate Number (`parseInteger`); combine (`checkNumber`) | Field rules: who may change it (no login, takes effect on Check or Enter, nothing saved) |
| Even/Odd rule | Field rules: apply the Even/Odd rule (`classifyParity`) | — (not a field) |

## Implementation-map coverage

| Map row | Created by |
|---|---|
| `even-odd-checker.html`, `vite.config.ts`, `main.tsx` (C7) | UI: Standalone page entry |
| `types.ts` | Field rules: define the outcome and parse types |
| `parseInteger()` (C1) | Field rules: validate Number |
| `classifyParity()` (C2) | Field rules: apply the Even/Odd rule |
| `checkNumber()` (C3) | Field rules: combine |
| `EvenOddChecker` (C4) | UI: empty (first use) state; extended by the later UI state, focus and test-ID tasks |
| `ResultMessage` (C5) | UI: result message |
| `messages` (C6) | UI: copy constants |
| Screen | UI tasks above |

## Blocked and open
None.
