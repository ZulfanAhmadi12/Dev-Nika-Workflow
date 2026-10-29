---
name: qa
description: Dev-Nika-Workflow QA (implementer stage) — automated gates, then E2E over the plan's QA scenarios after a "Go to QA" verdict, until QA passed.
argument-hint: "<plan number, e.g. 016>"
disable-model-invocation: true
---

# QA

This session runs **QA** as the implementer for plan: $ARGUMENTS

QA permission comes from the same plan's implementation permission (rule 1). The user typing this command with a plan number gives it if it was not given yet.

## Before anything

1. Read `${CLAUDE_PLUGIN_ROOT}/references/workflow-rules.md` (the portable rules; "rule N" below means its section 2).
2. Read `${CLAUDE_PROJECT_DIR}/.claude/workflow.md` (the **config**). If it does not exist, stop and follow `${CLAUDE_PLUGIN_ROOT}/references/config-interview.md`, using the template `${CLAUDE_PLUGIN_ROOT}/templates/workflow-config.template.md`.
3. Read the instruction files listed in config § Project. Chat in the config's chat language; write results in its docs language.
4. Before reading any PDF, DOCX, XLSX or PPTX, follow rules § 9 (Reading documents).

## 0. Supporting skills

Once the prerequisites hold, propose skills once (rules § 5), with reasons specific to this plan's scenarios:

- `design:accessibility-review` and `design:ux-copy`: when there are UI scenarios or user-facing copy;
- `superpowers:verification-before-completion`: before claiming QA passed;
- `superpowers:systematic-debugging`: proposed mid-way, when a scenario fails and the root cause is unclear.

## 1. Prerequisites

1. The plan's latest review says **Go to QA**, and the required findings are Fixed. Otherwise stop.
2. **E2E slot** (rule 11): only one plan runs E2E and resets the shared dev environment at a time. Check the slot queue where config § E2E environment says it is recorded. If another plan holds it, stop and report; never reset. Layer 1 below may still run.
3. This plan's implementer branch, `git rebase <integration branch>`, and re-check anything order-sensitive against the integration branch (for example migration timestamps).

## 2. Layer 1 — automated gates on the rebased commit

The gates in config § Commands, in their order, with the per-plan isolation from config § Test isolation, plus build when the config requires it for this kind of change, plus any extra gates the config lists.

## 3. Layer 2 — E2E

- Take the slot (record it), run the reset in config § E2E environment (the only reset allowed, and only now), then the demo setup in config § Demo setup runbook. Set the E2E run mode the config names and restart dev servers after changing build-time flags.
- **Tier A:** run **every** scenario in the plan, plus the review's Notes for QA. **Tier B:** run the affected scenarios and every negative/security scenario; list the ones not run and why in QA results.
- Use the browser and tools in config § Browser and manual QA and respect its known limits.
- **Steps only the user can do** (🔑 scenarios, and the kinds of steps the config lists) are **Needs manual verification**, never Passed. Collect them; do not ask one at a time. When you reach the point where you need them, send the user **one** message whose first line is exactly `NEEDS USER INPUT`, followed by: the demo setup state they start from, then every pending step numbered, each with plan/scenario ID, account, short steps and the expected result. Add the same items to the shared **Needs manual verification** list in the QA commit.
- Negative and security: check network traffic, logs and URLs for anything config § Non-negotiable rules forbids, and for data sent to a user who must not receive it.
- UI changes: keyboard/focus, light/dark, mobile width, copy language (with `design:accessibility-review` and `design:ux-copy` if approved).
- Follow config § E2E environment's test-data rule; never read secret paths.

## 4. Pause = commit

Whenever QA pauses — waiting for the E2E slot, waiting for `NEEDS USER INPUT` answers, or ending a turn with scenarios still open — commit the results so far in the plan's QA results (status **QA**, open scenarios marked "Not run yet" or "Needs manual verification") and record who holds the E2E slot. Uncommitted QA results are lost when a worktree is archived. This is the only bookkeeping commit allowed (rule 5).

## 5. Sorting QA findings

| Kind | Action | Re-review |
|---|---|---|
| **Implementation bug** (departs from a clear plan/contract) | Fix, add a regression test, re-run gates, the failed scenario, and the other scenarios in the same area | Only if the fix touches tier A or changes the approach |
| **Requirement or scenario problem** | Stop, write it under Open questions; the planner decides with the user | Yes |
| **Bug outside the plan's scope** | Not fixed here; record it in QA results for a new plan/backlog item | Follows the new plan |
| **Environment problem** (browser, database, flaky test) | Fix the environment or mark Needs manual verification; never change code to work around it | No |
| **Security finding** (protected data leaks, access control bypassed, data to a user who must not get it) | **Stop and report to the user**; the fix takes priority | Always |

- Attempt limit: the same scenario failing after two fixes, or a fix that needs a design change → stop and raise it to the planner.
- Find the root cause before fixing. If it is still unclear after one check, propose `superpowers:systematic-debugging` with the reason.
- A Passed result is valid only for the commit QA last ran on; any later fix means re-running the affected scenarios.

## 6. QA results (short)

Fill the plan's QA results: commit tested, one gate line, then a table `ID | Result | Short evidence` (one sentence per row). Long evidence (network captures, logs) is not copied into the plan; say where it was observed. Record QA findings with their kind and the fix commit.

## 7. Gate 4 — QA done

If `superpowers:verification-before-completion` was approved, run it before ticking this list.

- [ ] Required review findings are fixed; gates pass on the last commit.
- [ ] Every scenario is Passed, or Needs manual verification with a reason; none Failed.
- [ ] QA findings are sorted; those needing re-review were reviewed.
- [ ] Plan status **QA passed**, feature catalog updated, durable lessons in lessons learned, final diff read so unrelated user changes are intact.

## 8. Hand-off

QA results, plan status, catalog and fixes go into the same work commit (rule 5). Commit with plan/decision references, `git rebase <integration branch>`, confirm `git merge-base --is-ancestor <integration branch> HEAD`, never push. This branch touches code, so give the user the merge and push commands for the main checkout:

```
git merge --ff-only <branch>
git push origin <integration branch>
```

Then release the E2E slot and name the next plan in the queue. End with the report header (rules § 7).
