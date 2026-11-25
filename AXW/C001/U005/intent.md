# Intent — Even or Odd Number Checker

| | |
|---|---|
| **Project** | AXW |
| **Capability** | AXW-C001 |
| **Unit** | AXW-C001-U005 |
| **Requested by** | Story requester (Jira story "Develop an Even or Odd Number Checker") |
| **Agreed on** | 2026-11-25 |

## 1. Summary
This unit delivers a standalone single web page where a user enters one integer, presses a Check button and is told
whether the number is an "Even Number" or an "Odd Number". The page rejects empty input and input that is not a whole
number with a clear message, and a Clear button empties both the input and the result. Zero and negative integers are
valid. The page needs no login, stores nothing, and has no backend or database. It is a beginner training/demo
exercise that gives users a simple tool to check whether a number is even or odd.

## 2. Background
The request comes as a Jira story labelled `beginner`, `frontend` and `no-database`, in the Frontend component, with no
dependencies. The story reads: "As a user, I want to enter a number and check whether it is even or odd, so that I can
easily identify the number type." The requester confirmed it is a beginner training/demo exercise and that it lives on
its own standalone page. No login is required, and no other application links to it.

## 3. Problem
- **Who is affected:** Anyone who wants to check whether a number is even or odd (the story's "user").
- **What goes wrong today:** Users have no simple tool for checking whether a number is even or odd.
- **Impact:** Users can't quickly and reliably identify a number's type. As a training exercise, the team also has no
  small, fully specified feature to practise the delivery workflow on.
- **Evidence:** The Jira story's description, and the requester's confirmation that this is a beginner training/demo
  exercise in which users need a simple tool to check whether a number is even or odd.

## 4. Outcomes

| ID | Outcome | How we'll measure it | Baseline today | Target |
|---|---|---|---|---|
| O1 | The checker behaves exactly as the story's acceptance criteria describe | Number of the 7 acceptance criteria that pass QA | 0 of 7 (no checker exists) | 7 of 7 |
| O2 | Users get the correct Even/Odd answer for every valid integer | Share of valid integers tested (including zero and negatives) that show the correct result | No checker exists | 100% |

## 5. Scope

### In
| # | This unit covers | Serves |
|---|---|---|
| IN-1 | A standalone single page, with no login, where the user can enter an integer | O1 |
| IN-2 | A Check button that shows "Even Number" when the integer is divisible by 2 and "Odd Number" when it is not, with zero and negative integers treated as valid | O1, O2 |
| IN-3 | The message "Please enter a number" when Check is pressed with empty input | O1 |
| IN-4 | The message "Please enter a valid integer" when Check is pressed with invalid or decimal input | O1 |
| IN-5 | A Clear button that clears both the input and the result | O1 |

### Out
| # | Excluded | Why | Where it goes instead |
|---|---|---|---|
| OUT-1 | Storing data or keeping a history of checks | Not needed for the exercise; agreed with the requester | Not planned |
| OUT-2 | Any backend or database | The story is frontend-only, labelled `no-database` | Not planned |
| OUT-3 | Checking more than one number at a time | Only one number is checked at a time; agreed with the requester | Not planned |
| OUT-4 | Other number checks (prime, positive/negative, etc.) | Outside the story's need; agreed with the requester | Not planned |
| OUT-5 | Login, and links to or from other applications | The page is standalone; agreed with the requester | Not planned |

## 6. For requirements

| Topic | What the stakeholders said |
|---|---|
| What users need to do (each action) | Acceptance criteria, word for word: "1. User should be able to enter an integer. 2. User should be able to click the Check button. 3. If the number is divisible by 2, display "Even Number". 4. If the number is not divisible by 2, display "Odd Number". 5. If the input is empty, display "Please enter a number". 6. If the input is invalid or decimal, display "Please enter a valid integer". 7. User should be able to clear the input and result using the Clear button." |
| Who uses it, and who must not | "A user". No login; anyone who opens the page can use it. Nobody is excluded. |
| Business rules | A number divisible by 2 is even; otherwise it is odd. Zero is even. Negative integers follow the same divisibility rule. Only one number is checked at a time. Not stated: the largest and smallest integer accepted; whether surrounding spaces, a leading "+", leading zeros or exponent notation (e.g. "1e3") count as valid; whether pressing Enter also triggers Check; what happens to a shown result when the input is edited. |
| Information involved, and how sensitive it is (personal, salary, health…) | One number typed by the user. It isn't personal or sensitive and isn't stored. |
| Volumes and peaks (with who gave the figure) | Not stated. |
| Other systems or processes it depends on, and how reliable they are | None. The story lists "Dependencies: None"; there is no backend or database. |
| Money movement, or changes that can't be undone | None. Nothing is stored. |
| Constraints: legal, policy, deadline, technical | Frontend only (Component: Frontend; labels `frontend`, `no-database`). A standalone single page with no login. Deadline not stated. |
| Quality expectations: speed, availability, security, accessibility | Not stated. |

## 7. Glossary
| Term | Meaning in this unit |
|---|---|
| Integer | A whole number with no fractional part: positive, negative or zero |
| Even Number | An integer divisible by 2 with no remainder, including zero and negative integers such as -4 |
| Odd Number | An integer not divisible by 2, including negative integers such as -3 |
| Decimal | A number with a fractional part (e.g. 4.5); it isn't an integer and is rejected |
| Invalid input | Anything entered that isn't an integer, such as letters or symbols |
| Check button | The control the user presses to have the entered number judged |
| Clear button | The control that empties both the input and the shown result |
| Result | The message shown after Check: "Even Number", "Odd Number", "Please enter a number" or "Please enter a valid integer" |
| Standalone page | A single web page with no login that no other application links to |

## 8. Decisions made during clarification
| Question | Answer | Answered by (name or role) | Date |
|---|---|---|---|
| Where does the checker live? | A standalone single page, with no login and no links from other applications | Story requester | 2026-11-25 |
| What is the problem and who has it? | A beginner training/demo exercise; users need a simple tool to check whether a number is even or odd | Story requester | 2026-11-25 |
| How will success be measured? | O1: all 7 acceptance criteria pass QA (7/7). O2: 100% correct Even/Odd results for all valid integers tested | Story requester | 2026-11-25 |
| Are zero and negative integers valid? | Yes. Zero is even, and negative numbers follow the same divisibility rule | Story requester | 2026-11-25 |
| What is out of scope? | No data storage or calculation history; no backend or database; one number checked at a time; no other number checks (prime, positive/negative, etc.) | Story requester | 2026-11-25 |
