# Intent — Even or Odd Number Checker

| | |
|---|---|
| **Project** | AXW |
| **Capability** | AXW-C001 — Features |
| **Unit** | AXW-C001-U006 |
| **Requested by** | Stakeholder story "Develop an Even or Odd Number Checker" |
| **Agreed on** | 2026-11-25 |

## 1. Summary
A single-screen, frontend-only checker: the user enters a whole number, clicks **Check**, and sees "Even Number" or
"Odd Number". Empty or non-integer input gets a message instead, and **Clear** resets the screen. It is a beginner
training exercise for practising the spec-to-build workflow.

## 2. Background
Stakeholder story: "As a user, I want to enter a number and check whether it is even or odd, so that I can easily
identify the number type." Labels: `beginner`, `frontend`, `no-database`. Dependencies: none.

## 3. Problem
- **Who is affected:** The delivery team learning the spec-to-build workflow.
- **What goes wrong today:** There is no small, self-contained story to practise the workflow end to end.
- **Impact:** Learning the workflow on larger stories is harder, because the domain gets in the way.
- **Evidence:** Confirmed as a training exercise during clarification. The story's labels say the same.

## 4. Outcomes

| ID | Outcome | How we'll measure it | Baseline today | Target |
|---|---|---|---|---|
| O1 | Every input shows the correct message | Kinds of input (even, odd, empty, invalid) that show the correct message | 0 of 4 (no checker) | 4 of 4 |

## 5. Scope

### In
| # | This unit covers | Serves |
|---|---|---|
| IN-1 | Enter a whole number and check whether it is even or odd | O1 |
| IN-2 | Be told when the input is empty or not a whole number | O1 |
| IN-3 | Clear the input and the result to start again | O1 |

### Out
| # | Excluded | Why | Where it goes instead |
|---|---|---|---|
| OUT-1 | Storing input or results | Labelled `no-database` | Not planned |
| OUT-2 | Any server or API part | Frontend-only component | Not planned |
| OUT-3 | Changes to AXW-C001-U005 (even-odd-number-checker) | This story is specified as a separate unit | U005, unchanged |

## 6. For requirements

| Topic | What the stakeholders said |
|---|---|
| What users need to do (each action) | Enter a number, Check, Clear. |
| Who uses it, and who must not | Any user. No sign-in or roles. |
| Business rules | Messages, exact wording: divisible by 2 → "Even Number"; not divisible by 2 → "Odd Number"; empty → "Please enter a number"; invalid or decimal → "Please enter a valid integer". Zero and negative integers are valid. Not stated: spaces around the number, a leading "+", very large numbers, whether the result stays when the input is edited, whether Enter triggers Check. |
| Information involved, and how sensitive it is | One typed number. Not stored. Not sensitive. |
| Volumes and peaks | Not stated. |
| Other systems or processes it depends on | None. |
| Money movement, or changes that can't be undone | None. |
| Constraints: legal, policy, deadline, technical | Frontend only. No database. No deadline stated. |
| Quality expectations: speed, availability, security, accessibility | Not stated. |

## 7. Glossary
| Term | Meaning in this unit |
|---|---|
| Integer | A whole number: positive, negative or zero (e.g. -7, 0, 42) |
| Even / Odd | Divisible / not divisible by 2. Zero is even |
| Invalid input | Non-empty input that is not an integer, including decimals (e.g. 3.5, "abc") |
| Message | The text shown after Check: one of the four messages listed under Business rules |

## 8. Decisions made during clarification
| Question | Answer | Answered by | Date |
|---|---|---|---|
| U005 already covers this story. New unit? | Yes. U005 stays unchanged | User (BA) | 2026-11-25 |
| How is success measured? | The correct message for every input | User (BA) | 2026-11-25 |
| What is the problem? | Training exercise for the spec-to-build workflow | User (BA) | 2026-11-25 |
| Which integers are valid? | All, including zero and negatives | User (BA) | 2026-11-25 |
