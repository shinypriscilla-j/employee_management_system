# Design — Even or Odd Number Checker

| | |
|---|---|
| **Unit** | AXW-C001-U005 (even-odd-number-checker) |
| **Capability** | AXW-C001 |
| **Kind** | ui |
| **Inputs** | intent.md (5671c1b) · requirements.md (c42728e, approved by Story requester, 2026-11-25) |
| **Status** | Approved by Story requester, 2026-11-25 |
| **Author** | @architect (architect-unit-design) |

Everything marked **Proposed** is a draft decision for the accountable human to confirm.
Everything marked **UNRESOLVED** is not decided and must not be decided in code.

## 1. Approach

**Decision (proposed):** one self-contained React page, built entirely in the browser.
The page has its own HTML entry point in the host UI repo, so it is standalone: no router, no navigation, and no link to or from any other page (intent OUT-5).
All the logic is one pure function, `checkNumber(raw)`, which returns one of four outcomes: `even`, `odd`, `number_missing` or `not_an_integer`.
The function never converts the text to a number.
It validates the text against the Number field rule, then decides Even/Odd from the **last digit** of the text.
A whole number's parity depends only on its last digit, whatever its sign, its leading zeros or its length.
This gives the correct answer for integers of any length (R11, R12) with no number-size limit.
The component holds two pieces of state, the typed text and the current outcome, and renders the outcome as one message.

**Rejected alternative 1:** convert the text with `Number()` / `parseInt()` and test `n % 2`.
Rejected because JavaScript numbers lose precision above 2^53 − 1 (9,007,199,254,740,991).
`98765432109876543211` is stored as `98765432109876540000`, which would show "Even Number" and break R11 and outcome O2.
`BigInt(text)` would fix the size but still needs the same text validation, and it accepts forms the rule rejects (e.g. `" 0x1A"` style prefixes), so it adds a step without removing one.

**Rejected alternative 2:** an `<input type="number">`.
Rejected because browsers report an empty value for text they can't parse (e.g. `abc`, `1,000`) and accept `1e3` and `4.0` as numbers.
So R20–R22 would look like R16, and R19/R21 would pass as valid, breaking the `not_an_integer` rows.

No requirements issue to raise with the BA: every R-ID can be met on the stack.

## 2. Components & flows

### Components

| # | Component | Responsibility | R-IDs | Implementation |
|---|---|---|---|---|
| C1 | Integer parser | Applies the Number field rule to the typed text: strips spaces (U+0020) before and after; empty → `empty`; matches `^[+-]?[0-9]+$` → `integer` with its digits; anything else → `invalid` | R7–R10, R16–R23 | UI · `parseInteger()` · `src/features/even-odd-checker/logic/checkNumber.ts` (new) |
| C2 | Parity classifier | Given the digit string, returns `even` if its last character is 0, 2, 4, 6 or 8, otherwise `odd` | R2–R12 | UI · `classifyParity()` · `src/features/even-odd-checker/logic/checkNumber.ts` (new) |
| C3 | Number checker | Combines C1 and C2 into one `CheckOutcome`: `empty` → `number_missing`; `invalid` → `not_an_integer`; `integer` → C2's answer | R2–R12, R16–R23 | UI · `checkNumber()` · `src/features/even-odd-checker/logic/checkNumber.ts` (new) |
| C4 | Checker form | Holds `value` and `outcome`; runs C3 on submit (Check click or Enter); clears `outcome` when `value` changes; empties both on Clear; manages focus | R1, R13–R15, R24, R25 | UI · `EvenOddChecker` · `src/features/even-odd-checker/EvenOddChecker.tsx` (new) |
| C5 | Result message | Renders the message for an outcome inside an always-present status region, so screen readers announce it | R2–R12, R16–R23 | UI · `ResultMessage` · `src/features/even-odd-checker/components/ResultMessage.tsx` (new) |
| C6 | Copy | The page's user-visible strings, as one constants object | all | UI · `messages` · `src/features/even-odd-checker/messages.ts` (new) |
| C7 | Page entry | A standalone HTML entry that mounts C4 inside the MUI theme | R1 | UI · `even-odd-checker.html`, `src/features/even-odd-checker/main.tsx` (new); `vite.config.ts` (modified) |

### Number checker rule (C1–C3)
Source: requirements §2 Number field rule and Even/Odd rule.

```
checkNumber(raw):
  1. text = raw with leading and trailing U+0020 spaces removed   (R7)
  2. text = ""                        → number_missing             (R16, R17)
  3. text doesn't match ^[+-]?[0-9]+$ → not_an_integer             (R18–R23)
  4. last = last character of text
  5. last ∈ {0,2,4,6,8}               → even                       (R2, R4, R5, R7, R10, R12)
  6. otherwise                        → odd                        (R3, R6, R8, R9, R11)
```

- Only the ASCII space is stripped, because the rule says "spaces". A tab or non-breaking space makes the input invalid.
- `[0-9]` is the ASCII digits only. Other scripts' digits (e.g. `٤`) are `not_an_integer`.
- `-0`, `+0` and `000` are all zero and Even (R10).
- No length limit is applied anywhere (requirements Q1).

### Flow F1 — check a number (R1–R13, R16–R24)

1. The user types into the Number field. C4 stores the text as typed and clears any shown outcome (see F3).
2. The user clicks Check, or presses Enter in the Number field. Both submit the same `<form>` (R13).
3. C4 calls `checkNumber(value)` (C3) and stores the outcome. The typed text isn't changed (R18–R23 "the Number field still holds …").
4. C5 renders the outcome's message in the status region, which announces it.
5. Focus: for `even` / `odd`, focus stays where it was (the Check button or the Number field). For `number_missing` / `not_an_integer`, focus moves to the Number field, which is marked invalid and described by the message.

| Step fails | What the user observes |
|---|---|
| 2 — Check clicked twice, or double-clicked (R24) | The second submit computes the same outcome and replaces the state with an equal value. One message is shown; nothing stacks |
| 3 — the text is empty or only spaces | "Please enter a number" (`number_missing`); no Even/Odd shown (R16, R17) |
| 3 — the text isn't an integer | "Please enter a valid integer" (`not_an_integer`); no Even/Odd shown; the text stays (R18–R23) |
| 3 — an unexpected script error | N/A — because `checkNumber` is a total function over any string: every input reaches exactly one of the four outcomes and nothing can throw. The unit tests prove it |
| 4 — the same message is shown again (R24) | The status region's text doesn't change, so it isn't re-announced. This is accepted: nothing new appeared |
| any — the page's script failed to load | The browser shows a blank page. N/A to this design — because there is no server, and the requirements have no load-failure row; project baseline |

### Flow F2 — clear (R14, R25)

1. The user clicks Clear (a `type="button"`, so it never submits).
2. C4 sets `value` to `""` and `outcome` to `null`.
3. Focus moves to the Number field, ready for the next number.

| Step fails | What the user observes |
|---|---|
| 1 — nothing typed and nothing shown (R25) | Same state set again; the page stays as it was; focus moves to the Number field |

### Flow F3 — edit after a result (R15)

1. A result or message is shown.
2. The user changes the Number field (typing, deleting, pasting or cutting).
3. C4's change handler stores the new text and sets `outcome` to `null` in the same update, so the message disappears straight away.
4. Nothing more is shown until the next Check or Enter (F1).

| Step fails | What the user observes |
|---|---|
| 2 — the change leaves the text the same (e.g. select-all then retype the same digits) | The browser fires a change, so the message still disappears; Check shows it again. Accepted: the user edited the field |

## 3. API
N/A — because the unit has no API surface (`api = n/a`); everything happens in the browser (intent OUT-2).

## 4. Data
N/A — because the unit owns and stores no data (intent OUT-1, OUT-2).
The typed text and the outcome are React component state only: they're not written to local storage, session storage, cookies or the URL, and a reload starts empty.

### State & idempotency
- **State machine (C4 `outcome`):** `null` (nothing shown) → `even` | `odd` | `number_missing` | `not_an_integer` on submit; any of these → `null` on change or Clear; any of these → the newly computed outcome on submit.
  There is no terminal state.
  **Invariant:** at most one message is shown at a time. This is enforced by `outcome` being a single value, not a list, and by C5 rendering only that value.
- **Idempotency walk:** the only effect is "show the outcome for the current text". First submit, second click, double-click (R24), Enter followed by a click: each one computes `checkNumber(value)` from the same `value`, which is fixed before the call, and sets the same outcome. Result: one message.
- **Concurrency matrix:**

| At the same time | Outcome | Enforced by |
|---|---|---|
| Typing and a submit in the same event loop | React applies the updates in order; the submit uses the value committed at submit time | Application (React state updates) |
| The page open in two tabs | Independent pages; nothing is shared | — |

### Existing data
None — because this is a new page that stores nothing.

## 5. Contracts

| File | Type | Purpose | Machine-validated |
|---|---|---|---|
| — | — | — | — |

No contract files: there is no API (no `openapi.yaml`), no owned entity (no SQL or schema) and no events (no `asyncapi.yaml`).
The UI contract is this document's § UI: its states, copy, error mapping and test IDs.

## 6. UI

**Mockups:** none exist (confirmed by the requester, 2026-11-25). This section is the specification; the frame inventory below is what a designer should produce from it.

### Screens

| Screen | Entry | Purpose | API calls | R-IDs |
|---|---|---|---|---|
| Even or Odd Number Checker | `/even-odd-checker.html` (its own HTML entry in the host UI repo; served wherever that repo is deployed — D8) | Enter one integer and see whether it is even or odd | none | R1–R25 |

**Layout (one column, centred, maximum width 480 px; the same at every width down to 320 px)**

```
┌──────────────────────────────────────────────┐
│  Even or Odd Number Checker            (h1)  │
│                                              │
│  Number                                      │
│  ┌────────────────────────────────────────┐  │
│  │ 42                                     │  │
│  └────────────────────────────────────────┘  │
│                                              │
│  [ Check ]  [ Clear ]                        │
│                                              │
│  Even Number                  (status region)│
└──────────────────────────────────────────────┘
```

When the outcome is `number_missing` or `not_an_integer`, the Number field shows its error styling (red outline) and the message appears in the same status region in the error colour, preceded by an error icon. The text carries the meaning; colour and icon only support it.

**Frame inventory (for the designer):**
- F1: initial, empty, desktop
- F2: "Even Number" shown
- F3: "Odd Number" shown
- F4: "Please enter a number"
- F5: "Please enter a valid integer"
- F6: F2 at 320 px
- F7: F5 at 200% zoom

### Components

All new components live in `src/features/even-odd-checker/` in the host UI repo and use MUI 6.1.

| Component | Type | Inputs (type · required · default) | Events | States rendered |
|---|---|---|---|---|
| `EvenOddChecker` | new (page body) | — | — | initial · result shown · message shown |
| `ResultMessage` | new | `outcome: CheckOutcome \| null` (yes) · `id: string` (yes) | — | nothing (`null`) · even · odd · number_missing · not_an_integer |

`CheckOutcome = 'even' | 'odd' | 'number_missing' | 'not_an_integer'` (one `outcome` input, no booleans).

Reused MUI components:
- `Container`, `Paper` and `Stack` for layout
- `Typography` for the `h1` and the message text
- `TextField` for the Number field: `type="text"`, label "Number", `autoComplete="off"`, `error` when the outcome is a reason code, `inputProps={{ 'aria-describedby': <ResultMessage id> }}`
- `Button` for Check (`variant="contained"`, `type="submit"`) and Clear (`variant="outlined"`, `type="button"`)
- `ThemeProvider` and `CssBaseline` (the MUI default theme)
- `ErrorOutline` from `@mui/icons-material` for the error icon (Decision D4)

The Number field has no `inputMode`, so mobile keyboards keep the "-" and "+" keys (R5, R6, R8).
The `<form>` has `noValidate` and the field has no `required`, so the browser shows no messages of its own; the page's messages are the only ones (R16).

### States — Even or Odd Number Checker

| State | What the user sees | R-IDs |
|---|---|---|
| loading | Nothing until the script runs (static page, no data to fetch). N/A — because there is no asynchronous data | — |
| empty — first use | Heading, an empty Number field, Check and Clear, and an empty status region | R1 |
| empty — filtered | N/A — because there is no search or filter | — |
| populated | The typed text in the Number field; no result until Check | R1, R15 |
| partial | N/A — because nothing is loaded in parts | — |
| submitting | N/A — because the check is synchronous and finishes before the next frame; controls are never disabled | — |
| success | "Even Number" or "Odd Number" in the status region | R2–R13, R24 |
| error — recoverable | "Please enter a number" or "Please enter a valid integer"; the field is marked invalid and keeps the text; the user corrects it and presses Check | R16–R23 |
| error — terminal | N/A — because every message can be fixed by editing and checking again | — |
| disabled / unauthorized | N/A — because there is no login and nothing is restricted | — |
| offline — on load | N/A — because the page can't open without a connection; that's the browser's own page, and there is no server for this design to handle | — |
| offline — on submit | Works normally: the check needs no network | R2–R23 |
| session expiry mid-flow | N/A — because there is no session | — |
| cleared | Same as empty — first use | R14, R25 |
| edited after a result | The message disappears straight away | R15 |

### Validation & copy

Validation is C1–C3 (the Number field rule), run only on Check or Enter, never while typing (R15).

All strings live in `src/features/even-odd-checker/messages.ts` (English only, no i18n library — decided 2026-11-25).

| Key | English | Used for |
|---|---|---|
| `title` | Even or Odd Number Checker | `<h1>` and the document `<title>` |
| `numberLabel` | Number | Number field label |
| `check` | Check | Check button |
| `clear` | Clear | Clear button |
| `even` | Even Number | Outcome `even` (R2, R4, R5, R7, R10, R12) |
| `odd` | Odd Number | Outcome `odd` (R3, R6, R8, R9, R11) |
| `number_missing` | Please enter a number | Reason code `number_missing` (R16, R17) |
| `not_an_integer` | Please enter a valid integer | Reason code `not_an_integer` (R18–R23) |

The outcome messages are copied exactly from the acceptance criteria. No dates are shown.

### Reason code → UI
(No API errors exist; this maps each requirements §4 reason code to what the user sees.)

| Reason code | User sees | Field | Focus goes to |
|---|---|---|---|
| `number_missing` | "Please enter a number", error colour and icon | Marked invalid (`aria-invalid="true"`), text kept (empty) | The Number field |
| `not_an_integer` | "Please enter a valid integer", error colour and icon | Marked invalid, text kept as typed | The Number field |

### Accessibility (WCAG 2.2 AA)
- **Keyboard map:**
  - Tab order is Number field → Check → Clear.
  - Enter in the Number field submits (R13).
  - Enter or Space on a focused button activates it.
  - No Esc behaviour and no composite widgets.
- **Focus on entry:** the Number field is focused on load (`autoFocus`), because it's the page's only task.
- **Focus after submit, success:** stays where it was (the Check button, or the Number field after Enter).
- **Focus after an error:** the Number field (`aria-invalid="true"`, `aria-describedby` pointing at the message).
- **Focus after Clear:** the Number field.
- **Announcements:**
  - The status region (`role="status"`, i.e. `aria-live="polite"`) is in the DOM from the start, empty, so each new message is announced when it appears (requirements §5).
  - We chose a live region over moving focus to the result: focus stays at the controls the user needs next, and the standard's reason for moving focus doesn't apply because nothing follows the result.
- **Semantics:**
  - The page has one `<main>` containing a `<form>` labelled by the `<h1>`.
  - The field's accessible name is "Number".
  - The buttons' accessible names are "Check" and "Clear".
  - The error icon is decorative (`aria-hidden`).
- **Visual:**
  - MUI default theme contrast: ≥ 4.5:1 for text, including the error text, and ≥ 3:1 for the field outline.
  - Buttons and field are at least 36 px tall (≥ 24×24 CSS px).
  - Usable at 200% zoom, with no horizontal scroll at 320 px.
  - No animation beyond MUI's short, decorative button ripple.
  - Errors carry text and an icon, not colour alone.
- **Knowingly not covered:**
  - de and fr (out of scope, decided 2026-11-25).
  - Re-announcing an identical repeated result (F1 step 4 failure row).
  - Turning off the button ripple under `prefers-reduced-motion`: it's decorative, and WCAG 2.3.3 (animation from interactions) is level AAA, not AA.

### Test IDs

| `data-testid` | Element |
|---|---|
| `even-odd-page` | Page root (`<main>`) |
| `even-odd-title` | `<h1>` |
| `even-odd-input` | Number field `<input>` |
| `even-odd-check-button` | Check |
| `even-odd-clear-button` | Clear |
| `even-odd-result` | Status region (always present; empty when nothing is shown) |

The outcome is read from `even-odd-result`'s text. The same element also carries `data-outcome="even|odd|number_missing|not_an_integer"`, or no attribute when empty, so tests can assert the reason code without matching copy.

### Must NOT offer

| Affordance | Why |
|---|---|
| A history or list of past checks | OUT-1 |
| Saving the number across reloads (local storage, URL) | OUT-1; nothing is stored |
| More than one number field, or a list input | OUT-3 |
| Prime, positive/negative or any other check | OUT-4 |
| Login, navigation, or links to other pages | OUT-5 |
| Checking while typing (live result) | R15: a result appears only on Check or Enter |
| A numeric spinner, or a number-only keyboard that hides "-" / "+" | R5, R6, R8, and rejected alternative 2 |
| Browser validation pop-ups | R16: the page's own message is the only one |
| Disabling Check when the field is empty | R16 needs Check to work on empty input |

## 7. Implementation

**Host repo (Decision D8):** the page is built in the existing project UI repo, chosen when it is built. No new repo and no separate Employee Management UI application; the SoR store repo is not touched (requester, 2026-11-25). This design step creates no repo and no code. All paths below are relative to the host repo's root, which uses the D1 UI stack.

### 7.1 UI repo — the existing project UI repo (D8)

**Stack** (project decision D1, confirmed 2026-11-24; this unit's subset confirmed 2026-11-25):
- TypeScript 5.6 · React 18.3 · Vite 5.4 · MUI 6.1 (`@mui/material`, `@emotion/react`, `@emotion/styled`) · `@mui/icons-material` 6.1 (D4)
- Tests: Vitest 2.1 · React Testing Library 16 (`@testing-library/react`, `@testing-library/user-event` 14) · Playwright 1.48 · `@axe-core/playwright` 4.10
- Not used (decided 2026-11-25): React Router, TanStack Query, the generated API client, i18next

**Layout**
```
even-odd-checker.html                         standalone HTML entry (new)
vite.config.ts                                adds the entry to build.rollupOptions.input (modified)
src/features/even-odd-checker/
  main.tsx                                    mounts the page (new)
  EvenOddChecker.tsx                          C4 (new)
  EvenOddChecker.test.tsx                     component tests (new)
  messages.ts                                 C6 (new)
  types.ts                                    CheckOutcome, ParseResult (new)
  components/ResultMessage.tsx                C5 (new)
  logic/checkNumber.ts                        C1–C3 (new)
  logic/checkNumber.test.ts                   unit tests (new)
e2e/even-odd-checker.spec.ts                  Playwright, one test per R-ID (new)
e2e/even-odd-checker.a11y.spec.ts             axe checks (new)
```

**Implementation map**

| Design element | Class / function / component | File | New / modified | R-IDs |
|---|---|---|---|---|
| C7 Page entry | HTML document (`<title>`, `<div id="root">`, module script) | `even-odd-checker.html` | new | R1 |
| C7 Build entry | `build.rollupOptions.input.evenOddChecker` | `vite.config.ts` | modified | R1 |
| C7 Mount | `createRoot(...).render(<ThemeProvider><CssBaseline/><EvenOddChecker/></ThemeProvider>)` | `src/features/even-odd-checker/main.tsx` | new | R1 |
| Types | `CheckOutcome`, `ParseResult` | `src/features/even-odd-checker/types.ts` | new | R2–R23 |
| C1 Integer parser | `parseInteger(raw: string): ParseResult` | `src/features/even-odd-checker/logic/checkNumber.ts` | new | R7–R10, R16–R23 |
| C2 Parity classifier | `classifyParity(digits: string): 'even' \| 'odd'` | `src/features/even-odd-checker/logic/checkNumber.ts` | new | R2–R12 |
| C3 Number checker | `checkNumber(raw: string): CheckOutcome` | `src/features/even-odd-checker/logic/checkNumber.ts` | new | R2–R12, R16–R23 |
| C4 Checker form | `EvenOddChecker` (`handleChange`, `handleSubmit`, `handleClear`) | `src/features/even-odd-checker/EvenOddChecker.tsx` | new | R1, R13–R15, R24, R25 |
| C5 Result message | `ResultMessage` | `src/features/even-odd-checker/components/ResultMessage.tsx` | new | R2–R12, R16–R23 |
| C6 Copy | `messages` | `src/features/even-odd-checker/messages.ts` | new | all |
| Screen | Even or Odd Number Checker (`/even-odd-checker.html`) | as above | — | R1–R25 |

`ParseResult = { kind: 'empty' } | { kind: 'invalid' } | { kind: 'integer'; digits: string }`, where `digits` is the text without its sign.

**Tests**

| Kind | Location / naming | Covers |
|---|---|---|
| Unit | `src/features/even-odd-checker/logic/checkNumber.test.ts` (Vitest), table-driven over the R-ID example values | R2–R12, R16–R23, plus the edge values `+0`, `000`, a tab, `٤` and a 1,000-digit number |
| Component | `src/features/even-odd-checker/EvenOddChecker.test.tsx` (Vitest + RTL + user-event) | R13, R14, R15, R24, R25; focus after error and after Clear; `aria-invalid`; `data-outcome` |
| End-to-end | `e2e/even-odd-checker.spec.ts` (Playwright), one `test('R<n> …')` per R-ID; selects by `data-testid` only | R1–R25 on Chromium, Firefox and WebKit, plus mobile Chrome and Safari device profiles |
| Accessibility | `e2e/even-odd-checker.a11y.spec.ts` (`@axe-core/playwright`) | Initial, success and error states have no WCAG 2.2 AA violations; at 320 px there is no horizontal scroll |

## 8. Cross-cutting

| Concern | This unit |
|---|---|
| Tenant isolation | None — because there is no tenant data and no server |
| Authentication / authorization | None — because there is no login and anyone may use the page (intent decision, 2026-11-25) |
| Validation | C1–C3, client side only. There is no server to re-validate, because nothing leaves the browser |
| Error model | Two reason codes, `number_missing` and `not_an_integer`, shown as messages (§6). There is no HTTP error envelope, because there is no API |
| Data exposure | None — the typed number isn't personal data (requirements §5), isn't stored and isn't sent anywhere. No analytics or telemetry is added |
| Observability | None — because there is no server to record to (requirements §5). Support is diagnosed by reproducing the input; `data-outcome` exposes the decision |
| Performance | N/A per requirements §5. The check is synchronous. The page adds one bundle (React + MUI); the frontend standard's LCP < 2.5 s and INP < 200 ms apply as the default |
| Availability / rate limits | N/A — no server (requirements §5) |
| Browsers | Current desktop and mobile Chrome, Edge, Firefox and Safari (requirements §5). Covered by the Playwright projects in §7 |
| Accessibility | WCAG 2.2 AA (§6) |
| Migration / backfill | None — because nothing is stored |
| Feature flag | None — because this is a new standalone page with nothing in flight; not linking to it is enough |
| Failure recovery | None needed — the page has no state worth recovering; a reload starts empty |

## 9. Decisions & risks

### Decisions

| # | Decision | Alternative rejected | Why | Status |
|---|---|---|---|---|
| D1 | Project UI stack: TypeScript 5.6, React 18.3, Vite 5.4, MUI 6.1, Vitest, RTL, Playwright; without React Router, TanStack Query or the API client | A plain HTML/JS page | Follows the confirmed project stack | **Confirmed** — Story requester, 2026-11-25 |
| D2 | Decide Even/Odd from the last digit of the validated text | `Number(text) % 2`; `BigInt(text) % 2n` | Correct for any length (R11, R12); no precision loss; no second parser (§1) | Proposed |
| D3 | `type="text"` input with no `inputMode` | `type="number"`; `inputMode="numeric"` | `type="number"` hides invalid text (R19–R22); a numeric keypad on iOS has no "-" (R5, R6) | Proposed |
| D4 | Add `@mui/icons-material` 6.1 for the error icon | Text and colour only | WCAG: colour must not be the only signal; it's MUI's own icon package, at the same version as the stack | Proposed — new library |
| D5 | Standalone Vite HTML entry for the page | A route in an existing single-page app | No router (decided 2026-11-25); no link from other pages (OUT-5) | Proposed |
| D6 | Live status region for announcements; focus moves to the field only on error | Focus moves to the result after every Check | The user's next action is at the field; the region announces without moving them | Proposed |
| D7 | Strip only U+0020 spaces | `String.trim()` (all whitespace) | Requirements say "spaces"; nothing more is accepted than stated | Proposed |
| D8 | The page is built in the existing project UI repo, chosen when it is built; paths are relative to its root; it is served wherever that repo is deployed. The design is held in the SoR only — no `document.md` is written to an engineering repo | A new repo for the page; naming the repo now | No repo is created at design time; the design doesn't depend on the repo's name | **Confirmed** — Story requester, 2026-11-25 |

### Open items
No open items. No requirements questions or assumptions to carry: requirements §6 has none open.

### Risks (costliest to find after launch first)

| # | Risk | Response |
|---|---|---|
| K1 | The repo chosen at build time doesn't use the D1 stack (e.g. Angular, or no MUI), so §7 can't be built there as written | Mitigate: D8 requires the D1 UI stack; a different stack comes back through this skill as a design change |
| K2 | Someone "simplifies" the logic to `Number(x) % 2` and breaks long numbers | Mitigate: the unit tests include a 1,000-digit case and R11/R12 run in e2e |
| K3 | A screen reader doesn't re-announce an identical repeated message | Accept (§6 knowingly not covered) |

## 10. Requirement coverage

| R-ID | Design | UI | Implementation |
|---|---|---|---|
| R1 | C7; C4; F1 step 1 | empty — first use; populated | `even-odd-checker.html`; `main.tsx`; `EvenOddChecker.tsx` |
| R2 | C2/C3 rule 5 | success — "Even Number" | `checkNumber.ts`; `ResultMessage.tsx` |
| R3 | C2/C3 rule 6 | success — "Odd Number" | `checkNumber.ts`; `ResultMessage.tsx` |
| R4 | C3 rule 5 (`0`) | success | `checkNumber.ts` |
| R5 | C1 sign; rule 5 | success | `checkNumber.ts`; D3 |
| R6 | C1 sign; rule 6 | success | `checkNumber.ts`; D3 |
| R7 | C3 rule 1 | success | `checkNumber.ts` |
| R8 | C1 `+`; rule 6 | success | `checkNumber.ts`; D3 |
| R9 | C1 leading zeros; rule 6 | success | `checkNumber.ts` |
| R10 | C1 `-0`; rule 5 | success | `checkNumber.ts` |
| R11 | D2 last digit; rule 6 | success | `checkNumber.ts` |
| R12 | D2 last digit; rule 5 | success | `checkNumber.ts` |
| R13 | F1 step 2 (form submit) | keyboard map | `EvenOddChecker.tsx` |
| R14 | F2 | cleared | `EvenOddChecker.tsx` (`handleClear`) |
| R15 | F3 | edited after a result | `EvenOddChecker.tsx` (`handleChange`) |
| R16 | C3 rule 2; F1 failure row | error — recoverable | `checkNumber.ts`; `ResultMessage.tsx` |
| R17 | C3 rules 1–2 | error — recoverable | `checkNumber.ts` |
| R18 | C3 rule 3 | error — recoverable | `checkNumber.ts` |
| R19 | C3 rule 3 | error — recoverable | `checkNumber.ts` |
| R20 | C3 rule 3 | error — recoverable | `checkNumber.ts` |
| R21 | C3 rule 3 | error — recoverable | `checkNumber.ts` |
| R22 | C3 rule 3 | error — recoverable | `checkNumber.ts` |
| R23 | C3 rule 3 | error — recoverable | `checkNumber.ts` |
| R24 | F1 failure row; idempotency walk; invariant | success | `EvenOddChecker.tsx` (`handleSubmit`) |
| R25 | F2 failure row | cleared | `EvenOddChecker.tsx` (`handleClear`) |

**Field rules → where enforced:**
- Number field rule → C1 and C3 (`checkNumber.ts`).
- Even/Odd rule → C2.
- "Who may change it" (anyone, takes effect on Check or Enter, nothing saved) → C4 submit handler; § Data (no storage); no login (§8).

**Reason codes:**

| Code | Produced by | UI row |
|---|---|---|
| `number_missing` | C3 rule 2 | Reason code → UI: `number_missing` |
| `not_an_integer` | C3 rule 3 | Reason code → UI: `not_an_integer` |

**Requirements §5 differences → where applied:**
- Accessibility → § UI Accessibility.
- Browsers → §8 and the §7 Playwright projects.
- The N/A rows → §8.

Every component and the screen has an implementation-map row.

## 11. Change log

| Date | Version | By | Change |
|---|---|---|---|
| 2026-11-25 | v1 | @architect | First version; approved by Story requester; host repo settled as D8 |
