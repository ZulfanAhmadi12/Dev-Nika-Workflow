# Plan template

One file per task, written by the planner before implementation and continued by the implementer until QA passed. Write it in the docs language from `.claude/workflow.md` § Languages (translate the headings too if that language is not English, and keep the translation the same in every plan).

## File rules

- Name: `NNN-short-slug.md`, three-digit numbers in sequence, never reused. Only the planner takes a new number; re-check it after rebase.
- A plan is a work instruction, not a second source of truth. Requirements live in the requirements, decision records and contracts; the plan cites them and does not copy them.
- Section ownership:
  - **Planner:** Summary through QA scenarios, and the answers under Open questions.
  - **Implementer:** Status (after Ready for implementation, including who implements), ticking Tasks, Open questions, QA results.
- Plans at **QA passed** or **Deferred** stay in the folder as history.

## Statuses

| Status | Set by | Meaning |
|---|---|---|
| Draft | Planner | Requirements still being refined |
| Ready for implementation | Planner | Contracts updated; implementer may start once the user assigns this plan |
| Implementation | Implementer | In progress |
| Review | Implementer | Implementation and automated gates done; waiting for a reviewer session |
| Fixing | Implementer | Review said "Fix first", or a QA finding needs a fix |
| QA | Implementer | Review said "Go to QA"; E2E running (partial results committed at each pause) |
| QA passed | Implementer | Gate 4 met |
| Deferred | Planner | Decided by the user; reason written in Status |

## Template

````markdown
# Plan NNN: Title

## Status

| Field | Value |
|---|---|
| **Status** | Draft |
| **References** | requirement …, decision record …, contract … |
| **Prerequisites** | other plans that must be on the integration branch first, or "none" |
| **Implementation commits** | (implementer: who, branch, base, hash or range) |
| **Review** | (link to `<reviews dir>/NNN-slug-rK.md`) |
| **E2E slot** | (implementer: holding / waiting behind plan …/ released) |

## Summary

The problem solved and the expected outcome, two to four sentences.

## Scope

- In: …
- Out: …

## Requirements and acceptance criteria

1. … (requirement / decision reference)

**Edge cases:** empty data · draft or inactive items · revoked or expired access · … (each one either covered by an acceptance criterion above or explicitly out of scope)

## Contract changes

Contracts updated together with this plan, or "none".

## Design and affected files

Areas touched and the approach. Name the patterns or components that must be reused.

## Risks

Security-sensitive areas, data migration, performance, or "nothing special".

**Risk tier:** A / B — one-sentence reason (criteria in `.claude/workflow.md` § Risk tier A). Tier A requires `security-review` at review; tier B is limited to about 15 QA scenarios.

## Tasks

- [ ] T1 …
- [ ] T2 …

## QA scenarios

| ID | Group | Role / starting data | Steps | Expected result |
|---|---|---|---|---|
| Q1 | Normal | … | … | … |
| Q2 | Variant | … | … | … |
| Q3 | Edge case | … | … | … |
| Q4 | Negative & security | … | … | … |

Mark with 🔑 the scenarios that need a step only the user can do (config § Browser and manual QA).

## Open questions

| # | Question (implementer) | Answer (planner with the user) |
|---|---|---|
| P1 | … | … |

## QA results

**Commit tested:** …
**Automated gates:** lint ✔/✘ · type-check ✔/✘ · test ✔/✘ · build ✔/✘/n.a.

| ID | Result | Short evidence |
|---|---|---|
| Q1 | Passed / Failed / Needs manual verification / Not run yet | … |

QA findings:

| # | Kind | Scenario | Symptom | Action | Re-review |
|---|---|---|---|---|---|
| 1 | Implementation bug / Requirement / Out of scope / Environment / Security | Q… | … | … | Yes/No |
````
