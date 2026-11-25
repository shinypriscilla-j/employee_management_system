# Requirements — Even or Odd Number Checker

| | |
|---|---|
| **Unit** | AXW-C001-U005 |
| **Parent feature** | AXW-C001 |
| **Surface** | UI |
| **Status** | Ready |
| **Approved by** | Story requester, 2026-11-25 |

Non-functional and UX behaviour follow the project quality baseline unless §5 says otherwise.

## 1. Story
As a **user**, I want to **enter a number and check whether it is even or odd**, so that **I can easily identify the number type**.

## 2. Field rules

| Field | Rule | Who may change it |
|---|---|---|
| Number | Required when Check is pressed. Must be an integer: an optional leading "+" or "-" followed by one or more digits 0–9, with no other characters. Spaces before and after are ignored. Leading zeros are allowed ("007" is 7). "-0" is zero. Any number of digits is accepted; there is no largest or smallest value. Not accepted: a decimal point (including "4.0"), exponent notation ("1e3"), thousands separators ("1,000"), a lone "+" or "-", letters or other symbols. Input that is empty or only spaces counts as empty. Source: acceptance criteria 1, 5, 6; decisions of 2026-11-25. | Anyone who opens the page; no login. Takes effect when Check is pressed (or Enter is pressed in the Number field). Nothing is saved; the value lasts only while the page is open. |

**Even/Odd rule:** an integer divisible by 2 with no remainder is "Even Number"; otherwise it is "Odd Number". Zero is even. Negative integers follow the same rule. Source: acceptance criteria 3, 4; decision of 2026-11-25.

## 3. Acceptance criteria

### Happy path
| R-ID | Given | When | Then |
|---|---|---|---|
| R1 | The page is open and the Number field is empty | I type "42" | The Number field shows "42"; no result is shown yet |
| R2 | The Number field holds "42" | I click Check | The result "Even Number" is shown |
| R3 | The Number field holds "7" | I click Check | The result "Odd Number" is shown |
| R4 | The Number field holds "0" | I click Check | The result "Even Number" is shown |
| R5 | The Number field holds "-4" | I click Check | The result "Even Number" is shown |
| R6 | The Number field holds "-3" | I click Check | The result "Odd Number" is shown |
| R7 | The Number field holds " 42 " (spaces before and after) | I click Check | The result "Even Number" is shown |
| R8 | The Number field holds "+7" | I click Check | The result "Odd Number" is shown |
| R9 | The Number field holds "007" | I click Check | The result "Odd Number" is shown |
| R10 | The Number field holds "-0" | I click Check | The result "Even Number" is shown |
| R11 | The Number field holds "98765432109876543211" | I click Check | The result "Odd Number" is shown |
| R12 | The Number field holds "98765432109876543210" | I click Check | The result "Even Number" is shown |
| R13 | The Number field holds "7" and has focus | I press Enter | The result "Odd Number" is shown, exactly as if I had clicked Check |
| R14 | The Number field holds "42" and the result "Even Number" is shown | I click Clear | The Number field is empty and no result or message is shown |
| R15 | The Number field holds "42" and the result "Even Number" is shown | I change the Number field to "427" | The result disappears straight away; no result or message is shown until I press Check again. The same applies when a message from R16–R23 is shown. |

### Failure cases
| R-ID | Class | Given | When | Then |
|---|---|---|---|---|
| R16 | input | The Number field is empty | I click Check | I'm told "Please enter a number" (`number_missing`); no Even/Odd result is shown |
| R17 | input | The Number field holds only spaces ("   ") | I click Check | I'm told "Please enter a number" (`number_missing`); no Even/Odd result is shown |
| R18 | input | The Number field holds "4.5" | I click Check | I'm told "Please enter a valid integer" (`not_an_integer`); no Even/Odd result is shown; the Number field still holds "4.5" |
| R19 | input | The Number field holds "4.0" | I click Check | I'm told "Please enter a valid integer" (`not_an_integer`); no Even/Odd result is shown; the Number field still holds "4.0" |
| R20 | input | The Number field holds "abc" | I click Check | I'm told "Please enter a valid integer" (`not_an_integer`); no Even/Odd result is shown; the Number field still holds "abc" |
| R21 | input | The Number field holds "1e3" | I click Check | I'm told "Please enter a valid integer" (`not_an_integer`); no Even/Odd result is shown; the Number field still holds "1e3" |
| R22 | input | The Number field holds "1,000" | I click Check | I'm told "Please enter a valid integer" (`not_an_integer`); no Even/Odd result is shown; the Number field still holds "1,000" |
| R23 | input | The Number field holds only "-" | I click Check | I'm told "Please enter a valid integer" (`not_an_integer`); no Even/Odd result is shown; the Number field still holds "-" |
| R24 | repetition | The Number field holds "42" and the result "Even Number" is shown | I click Check again (or double-click it) | Exactly one result, "Even Number", is shown; nothing is repeated or stacked |
| R25 | state | The page is open, the Number field is empty and nothing is shown | I click Clear | The page stays as it was: Number field empty, no result or message shown |

**Not applicable:** authorization (the page has no login and anyone may use it — decided 2026-11-25), concurrency (one person on their own page; nothing is shared), dependency (the page depends on no other system), partial failure (nothing is saved or sent anywhere), ordering (no events or messages are exchanged), time (no rule depends on dates, times or time windows).
**Not decided:** None.

## 4. Reason codes

| Code | Meaning | R-ID |
|---|---|---|
| `number_missing` | Check was pressed with nothing entered (empty or only spaces); the user is told "Please enter a number" | R16, R17 |
| `not_an_integer` | Check was pressed with something that isn't an integer (decimal, letters, symbols, exponent, separators, a lone sign); the user is told "Please enter a valid integer" | R18, R19, R20, R21, R22, R23 |

## 5. Differences from the project baseline
The project baseline does not exist yet. Until it does, this unit states its own needs, agreed with the Story requester on 2026-11-25:

| Category | This unit requires | Instead of (baseline) | Source |
|---|---|---|---|
| accessibility | Everything can be done with the keyboard alone (typing, Check, Enter, Clear). Each result or message is announced by screen readers when it appears. | No baseline recorded | Story requester, 2026-11-25 |
| supported browsers | Current desktop and mobile versions of Chrome, Edge, Firefox and Safari | No baseline recorded | Story requester, 2026-11-25 |
| availability, latency, throughput, surge, rate limits | N/A — everything happens on the page with no server | No baseline recorded | Story requester, 2026-11-25 |
| authorization, tenant isolation | N/A — no login; anyone who opens the page may use it | No baseline recorded | Intent decision, 2026-11-25 |
| audit, observability | N/A — nothing is recorded and there is no server to record to | No baseline recorded | Intent (no backend or database) |
| data classification, retention and deletion | The typed number is neither personal nor special-category data; it isn't stored, so nothing needs retaining or deleting | No baseline recorded | Intent |
| idempotency, concurrency | Covered by R24; nothing is shared between users | No baseline recorded | Failure sweep |
| migration and backfill, feature flag | N/A — no existing data and nothing to switch off | No baseline recorded | Intent (new standalone page) |

## 6. Open questions and assumptions

| # | Item | Blocking? | Owner | Answer |
|---|---|---|---|---|
| Q1 | How large or small can the number be? | No | Story requester | No limit; a whole number of any length gets the correct answer (R11, R12). Answered 2026-11-25. |
| Q2 | Which typed forms count as a valid integer? | No | Story requester | Spaces before/after, a leading "+", leading zeros and "-0" are accepted. "4.0", "1e3", "1,000", a lone sign, letters and symbols are invalid. Only spaces counts as empty (Number field rule; R7–R10, R17–R23). Answered 2026-11-25. |
| Q3 | Does Enter in the Number field act like Check? | No | Story requester | Yes (R13). Answered 2026-11-25. |
| Q4 | What happens to a shown result when the number is edited? | No | Story requester | It disappears as soon as the input changes; a new one appears only on Check (R15). Answered 2026-11-25. |
| Q5 | Accessibility, browsers and speed? | No | Story requester | Keyboard-operable, results announced to screen readers; current desktop and mobile Chrome, Edge, Firefox and Safari; speed, availability and volume N/A (§5). Answered 2026-11-25. |

No assumptions.

**Out of this story:** Storing data or keeping a history of checks; any backend or database; checking more than one number at a time; other number checks (prime, positive/negative, etc.); login, and links to or from other applications.
