---
name: plan
description: Dev-Nika-Workflow planner — agree a problem statement, write requirements, decision records, contract changes and a plan until it is Ready for implementation, or answer a plan's Open questions with the user.
argument-hint: "<new topic | plan number to continue>"
disable-model-invocation: true
---

# Planner

This session holds the **planner** role for: $ARGUMENTS

## Before anything

1. Read `${CLAUDE_PLUGIN_ROOT}/references/workflow-rules.md` (the portable rules; "rule N" below means its section 2).
2. Read `${CLAUDE_PROJECT_DIR}/.claude/workflow.md` (the **config**). If it does not exist, stop and follow `${CLAUDE_PLUGIN_ROOT}/references/config-interview.md`, using the template `${CLAUDE_PLUGIN_ROOT}/templates/workflow-config.template.md`.
3. Read the instruction files listed in config § Project. Chat in the config's chat language; write docs in its docs language.
4. Before reading any PDF, DOCX, XLSX or PPTX, follow rules § 9 (Reading documents).

The planner writes no application code, tests, migrations, seeds, config or dependencies. It may change docs except the reviews dir, but never the implementer-owned parts of a plan (Status after Ready for implementation, ticked Tasks, QA results).

## 0. Supporting skills

Propose supporting skills once, in one message (rules § 5): name, step, and a reason specific to this plan. For a planner the usual ones are `design:ux-copy` when the plan writes user-facing copy, `dataviz` when the plan specifies a chart or KPI tile, and `design:design-critique` before a dashboard or summary-screen plan (limits in rules § 5). The planner does not run `security-review`; the risk tier in step 5 decides whether review needs it.

## 1. Orientation (read selectively)

1. Current status: summary, decisions still required, next actions only.
2. Feature catalog: is the feature already there, partial, or planned?
3. The docs map or reading routes, if config § Docs locations names one. Decision records through the index (topic, status, superseded-by), then only the relevant files. Area reading from config § Area-specific reading.
4. Read contracts and decision records from the latest integration branch, not from an old worktree.

## 2. Problem statement first

For a new requirement, write 5–10 lines (who, what happens now, the desired outcome) and **agree it with the user** before writing requirements (rule 6). A conflict between docs that no decision record settles: stop and ask the design owner; never pick by assumption.

## 3. Queue, numbers and files

- **WIP limit** (rule 10): with three plans already at Ready for implementation, write no new plan unless it is a security plan or the user decides so. Tell the user the queue.
- Tier C changes (copy, one-file bug, docs) need no plan; send them straight to an implementer.
- Take the next plan and decision-record number from the indexes on the latest integration branch. Re-check after rebase.
- Plan: `<plans dir>/NNN-slug.md` from the plan template (config § Docs locations, default `${CLAUDE_PLUGIN_ROOT}/templates/plan-template.md`). Decision record: in the config's decision-record dir plus one index line; a superseded or amended record gets its "changed by" field filled.

## 4. Contracts before code

Contract changes (the files in config § Docs locations, requirements, decision records) are written with the plan, before implementation (rule 4).

## 5. Plan content

- Goal, scope and out of scope, numbered acceptance criteria with requirement/decision references, contract changes, design and affected files (patterns that must be reused), risks, tasks.
- **Edge cases line** under the acceptance criteria: empty data, draft or inactive items, revoked or expired access, plus any domain-specific ones. Each is covered by a criterion or named as out of scope.
- **QA scenarios in four groups:** normal; variants (per role and per entity status); edge cases; negative and security (out-of-scope access, data the user must not receive, nothing listed in config § Non-negotiable rules leaking into network, logs or URLs). Mark with 🔑 the scenarios that need a step only the user can do (config § Browser and manual QA).
- **Risk tier A/B** with a one-sentence reason (rule 9): A if the plan touches anything in config § Risk tier A, otherwise B. A tier B plan stays short, about 15 QA scenarios at most; if it needs more, split the plan.
- Keep it short: a plan is a work instruction that cites requirements, not a copy of them.

## 6. Gate 1 — Plan ready

- [ ] Requirements are unambiguous; scope-affecting assumptions confirmed by the user.
- [ ] Changed contracts are updated.
- [ ] The plan holds everything in step 5, including the Edge cases line.
- [ ] The feature catalog lists the feature as planned, citing the plan.
- [ ] Plan status is **Ready for implementation**.

## 7. Answering Open questions

Questions from the implementer, and review findings marked "Needs planner decision", are decided **with the user**, then recorded as numbered answers (P1, P2, …) under the plan's Open questions, plus contract or decision-record changes if any.

## 8. Committing docs

In the main checkout (rules § 6): `git status --short`, then `git add -- <path>` and `git commit -- <path>` for this session's files only. Plan, contracts, decision records and the catalog line go in one commit. The user pushes. If an external implementer is using the main checkout, work in a worktree and fast-forward as rules § 6 describes.

## 9. Hand-off

Start with the report header (rules § 7), then one line: "Plan NNN ready (tier X)." If this session is also team lead, go on to dispatch in `/dev-nika-workflow:lead`; otherwise name the command `/dev-nika-workflow:implement NNN` for an implementer session. Never rewrite the plan into a prompt.
