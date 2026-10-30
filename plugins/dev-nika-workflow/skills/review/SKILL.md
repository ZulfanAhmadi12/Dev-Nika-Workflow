---
name: review
description: Dev-Nika-Workflow reviewer — independently review one plan's implementation against the plan, the contracts and the project rules, then write the review file with a verdict.
argument-hint: "<plan number, e.g. 019>"
disable-model-invocation: true
---

# Reviewer

This session holds the **reviewer** role for plan: $ARGUMENTS

## Before anything

1. Read `${CLAUDE_PLUGIN_ROOT}/references/workflow-rules.md` (the portable rules; "rule N" below means its section 2).
2. Read `${CLAUDE_PROJECT_DIR}/.claude/workflow.md` (the **config**). If it does not exist, stop and follow `${CLAUDE_PLUGIN_ROOT}/references/config-interview.md`, using the template `${CLAUDE_PLUGIN_ROOT}/templates/workflow-config.template.md`.
3. Read the instruction files listed in config § Project. Chat in the config's chat language; write the review in its docs language.
4. Before reading any PDF, DOCX, XLSX or PPTX, follow rules § 9 (Reading documents).

The reviewer writes only the reviews dir. It never changes code, requirements, decision records, contracts or the plan, and never applies fixes. Findings that are really a requirement or contract problem are marked **"Needs planner decision"**.

## 1. Independence (check first)

- This session must **not** be the session that wrote this plan's code, including that session's subagents. If it did, stop and ask the user to open a new reviewer session.
- This session does not review a plan it wrote as planner. If it did, say so and ask the user to confirm.
- Record the role and the reason for independence in the review's **Role** field.

## 1b. Depth by risk tier

- **Tier A:** all of steps 3–4, propose `security-review`, run the focused tests for the sensitive area.
- **Tier B:** short review: check every acceptance criterion and the Edge cases line against code and tests, the project's Forbidden actions and Non-negotiable rules, and the UI checklist if any. The implementer already ran `code-review`; read its result if recorded, do not repeat it. If the range touches tier A (config § Risk tier A), raise it to A, name the files, and continue as a tier A review.
- An old plan without a risk tier: rate it with the same criteria and say so in the **Role** field.

## 2. Prepare the range

1. This session's own worktree on a branch per the config's reviewer branch pattern. Make sure the implementation commits recorded in the plan's Status are present: `git rebase <integration branch>`, or check out the implementer's branch detached.
2. Review range = the implementation commits in the plan's Status. Round K = previous round + 1; read the previous round's review and its Status column.

## 3. Proposed skills and automated first pass

Propose to the user once (rules § 5), each with a reason specific to this range, then wait for approval:

- `code-review` (tier A; for B the implementer already ran it) at a level chosen from the range's size and risk (e.g. `medium` for a small UI change, `high` for a business flow, `xhigh` for security-sensitive code). Give the commit/file count as the reason for the level.
- `security-review` for tier A, including a B plan raised to A in step 1b; name the files that make it necessary.
- `design:accessibility-review` when the range changes screens or interactive components.
- `design:design-critique` when the range adds or reworks a dashboard or information-dense screen; translate its grades into the review's severity scale (rules § 5).

Skill output is **material, not findings**: verify every item against the code before it goes into the review file. A declined skill is replaced by the same checks done by hand in step 4.

## 4. Manual review

Read every commit in the range against:

- the plan's acceptance criteria, its Edge cases line, and the P-n answers under Open questions;
- the contracts and decision records the plan cites;
- config § Non-negotiable rules and § Forbidden actions, and the workflow rules;
- config § Review checklist additions (for example route guard order, audit trail without secrets, UI states and language);
- tests: behaviour and material failure paths covered; no skipped tests and no edited fixtures.

For tier A areas: run the focused test files for that area with the reviewer isolation in config § Test isolation and record the result. The full gate stays the implementer's job; record the implementer's numbers as claims.

## 5. Review file

`<reviews dir>/NNN-slug-rK.md` from the review template (config § Docs locations, default `${CLAUDE_PLUGIN_ROOT}/templates/review-template.md`). Every finding has file:line, the problem, the affected rule/requirement (workflow rule number, non-negotiable rule number, decision record, requirement, contract), severity (Critical/High/Medium/Low), suggested fix, and Status = Open. Never copy secrets, credentials or protected data. Include **Notes for QA** for the scenarios the findings affect.

## 6. Gate 3 — Review done

- [ ] Every commit in the range has been read.
- [ ] Every finding is complete (step 5).
- [ ] Previous round findings are marked closed or still open.
- [ ] Verdict: **Go to QA**, or **Fix first** if a Critical/High finding is open. Medium/Low findings that may be carried are named.

## 7. Commit and hand-off

Commit only the reviews dir, `git rebase <integration branch>`, confirm `git merge-base --is-ancestor <integration branch> HEAD`, then fast-forward as docs-only per rules § 6 if the reviews dir is in the config's docs-only paths; otherwise hand the merge to the user.

End with the report header (rules § 7), then one line for the user:

- Fix first → "Run `/dev-nika-workflow:implement NNN fix rK` in the implementer session."
- Go to QA → "Run `/dev-nika-workflow:qa NNN` in the implementer session when the E2E slot is free."
- Any "Needs planner decision" → name the finding numbers for the planner session.
