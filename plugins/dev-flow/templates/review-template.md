# Review template

Findings of a review of one plan's implementation. A review reports findings and never applies them; the implementer fixes. Write it in the docs language from `.claude/workflow.md` § Languages.

## File rules

- Name: `NNN-slug-rK.md`, where `NNN-slug` matches the plan and `K` is the round (r1, r2, …). A new round is a new file; old round files are not changed except the findings' **Status** column.
- Every finding has file and line, the problem, the affected rule or requirement, severity and a suggested fix.
- Never copy secrets, credentials, personal data or other material listed in config § Non-negotiable rules into a review, not even as sample data.
- Written by a **reviewer** session that did not write the code (including through that session's subagents) and did not write the plan. The reviewer does not change requirements, decision records, contracts or the plan. Findings that need a requirement or contract decision are marked **"Needs planner decision"**.
- For changes in tier A areas, the reviewer runs the focused tests for that area and writes the result under Notes for QA. The full gate stays the implementer's job.
- The implementer changes only the **Status** column (Open → Fixed in `<commit>` / Not fixed: reason / Accepted (planner decision)). The reviewer verifies it next round.

## Severity

| Severity | Meaning | Example |
|---|---|---|
| Critical | Breaks a non-negotiable rule, leaks data, or bypasses access control | Secret in a log; endpoint without its scope check |
| High | Wrong behaviour against a requirement, or data loss | Status not updated after approval |
| Medium | Edge case, UI state, or missing test | No empty state; failure path untested |
| Low | Tidiness, naming, copy | Comment repeats the code |

Verdict **Fix first** while a Critical or High finding is open; otherwise **Go to QA**, naming the Medium/Low findings that may be carried.

## Template

````markdown
# Review NNN-rK: Plan title

| Field | Value |
|---|---|
| **Plan** | [`NNN-slug`](../plans/NNN-slug.md) |
| **Commits reviewed** | `<base>..<tip>` (branch, tip SHA) |
| **Date** | YYYY-MM-DD |
| **Role** | Reviewer; independence: did not write this code or plan because … |
| **Risk tier** | A / B (raised from B because … if applicable) |
| **Verdict** | Go to QA / Fix first |

## Summary

Two or three sentences: fit with the plan and the main risk.

## Findings

| # | Severity | File:line | Problem | Rule / requirement | Suggested fix | Status |
|---|---|---|---|---|---|---|
| 1 | High | `src/…:42` | … | workflow rule 3 / non-negotiable 2 / decision … / requirement … | … | Open |

## Previous round findings

(r2 onward: status of every finding from the previous round after verification)

## Notes for QA

Areas that need extra attention during E2E, focused tests run and their result, or "none".
````
