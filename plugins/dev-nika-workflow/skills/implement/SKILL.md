---
name: implement
description: Dev-Nika-Workflow implementer — implement a plan at Ready for implementation, or fix the findings of a review round, until it is ready for an independent review.
argument-hint: "<plan number, e.g. 019> [fix rK]"
disable-model-invocation: true
---

# Implementer

This session holds the **implementer** role for plan: $ARGUMENTS

The user typing this command with a plan number gives implementation permission **for that plan only**, including review fixes and its QA until QA passed (rule 1). Without a plan number, stop and ask. The permission does not cover any other plan.

## Before anything

1. Read `${CLAUDE_PLUGIN_ROOT}/references/workflow-rules.md` (the portable rules; "rule N" below means its section 2).
2. Read `${CLAUDE_PROJECT_DIR}/.claude/workflow.md` (the **config**). If it does not exist, stop and follow `${CLAUDE_PLUGIN_ROOT}/references/config-interview.md`, using the template `${CLAUDE_PLUGIN_ROOT}/templates/workflow-config.template.md`.
3. Read the instruction files listed in config § Project. Chat in the config's chat language; write docs in its docs language.
4. Before reading any PDF, DOCX, XLSX or PPTX, follow rules § 9 (Reading documents).

The implementer never redefines requirements and never edits requirements, decision records or the contract files in config § Docs locations. In docs it changes only the plan's Status, Tasks, Open questions and QA results; the Status column of review findings; the shared status files; and the "other docs" the config allows.

## 0. Supporting skills

After reading the plan (step 1) and before writing code, propose supporting skills once, in one message (rules § 5), each with its step and a reason specific to this plan. Implementer candidates:

- `superpowers:test-driven-development`: bug fixes, regression tests that must fail first, or security acceptance criteria;
- `superpowers:systematic-debugging`: proposed mid-way, when a test fails and the root cause is unclear;
- `superpowers:verification-before-completion`: before claiming gate 2;
- `code-review`: before hand-off for tier B plans and tier C changes (rule 9);
- `dataviz`: before writing chart or KPI-tile code, with the palette taken from the project's design system (rules § 5).

Use only the approved ones. Without approval, do the same steps by hand.

## 1. Prerequisites

**Tier C (no plan):** when the user asks directly for a fast-path change, skip the plan steps; make the change, run the automated gates and `code-review` (if approved), and commit with a message that says "fast path" and why. If it turns out to touch tier A (config § Risk tier A), stop and ask for a plan.

1. Open `<plans dir>/NNN-*.md` and note its **Risk tier** (an old plan without one: rate it with config § Risk tier A and tell the user). If the work touches tier A on a tier B plan, stop and tell the planner to raise the tier. Status must be **Ready for implementation** (new work) or **Fixing** (review fixes). Otherwise stop and report.
2. Check the plan's Prerequisites (for example other plans that must already be on the integration branch).
3. For fixes: read the latest review `<reviews dir>/NNN-*-rK.md` and work the findings the verdict requires.

## 2. Environment

- A worktree under the config's worktree dir, on a branch named per the config's implementer branch pattern, from the latest integration branch. If the plan already has a branch from an external implementer (config § External implementer), branch from its tip, then `git rebase <integration branch>` (rules § 6).
- `git status --short`: existing changes belong to the user; keep them.
- Tests use the per-plan isolation in config § Test isolation. Never touch the shared dev environment in config § E2E environment; resets happen only while holding the E2E slot in `/dev-nika-workflow:qa`.
- Plan status (who, base, **Implementation**) is updated in the first code commit, not a separate docs commit (rule 5).

## 3. Read before writing code

- The plan with every P-n answer under Open questions; the contracts it cites.
- Lessons learned: only the section for this area.
- The rows of config § Area-specific reading that match the files you will touch, and any build notes in config § Branches.
- Mechanical searches may go to the `dev-nika-workflow:code-scout` subagent; searches in tier A areas stay on the session model (`Explore`).
- State scope-affecting assumptions. If a requirement is unclear, contradictory or impossible: write it under the plan's **Open questions** and stop on that item.

## 4. During implementation

1. The smallest change that meets the acceptance criteria; keep existing patterns and contracts. Nothing outside scope.
2. Validate input at the boundary, enforce authorization on the server side, use transactions for atomic mutations. Follow every rule in config § Non-negotiable rules.
3. Bug fixes and security acceptance criteria: write the regression test first and prove it fails before the fix (with `superpowers:test-driven-development` if approved).
4. Never weaken types, lint rules, tests or security controls to get a gate green. No new skipped test, type-error suppression or untyped escape hatch without a written reason.
5. Follow the test conventions in config § Test isolation.
6. Commit per meaningful step; messages name the plan number and the requirement/decision reference.

## 5. Gates and checks

1. In the order config § Commands gives (typically lint → type-check → test), plus build when the config says it is required for this kind of change.
2. Any extra gates the config lists (for example test vectors). Never edit fixtures.
3. Read the diff: no secrets, credentials, personal data, debug logs, or anything config § Non-negotiable rules forbids.
4. Pre-existing failures or unavailable tools are reported, never hidden.

## 6. Gate 2 — Implementation done

If `superpowers:verification-before-completion` was approved, run it before ticking this list: every tick must rest on output of a command run just now, not memory.

- [ ] Acceptance criteria met without side changes; the plan's Edge cases are handled or explicitly out of scope.
- [ ] Tests cover the behaviour and the material failure paths.
- [ ] Relevant gates pass.
- [ ] UI and area checks from config § Area-specific reading and § Review checklist additions are met.
- [ ] Plan Status and Tasks, feature catalog and current status match reality (closed items removed from current status).
- [ ] Commits and the commit range are recorded in the plan's Status; for fixes, every review finding's Status column is filled (Fixed in `<commit>` / Not fixed: reason).

## 7. Hand-off

Plan status **Review**, the commit range, Tasks and the feature catalog go into the last code commit (no separate "record the commit" commit; the range may be written as `<base>..HEAD` of this branch). Then `git rebase <integration branch>` and confirm `git merge-base --is-ancestor <integration branch> HEAD`. Never push.

End with the report header (rules § 7), then one line for the user: "Plan NNN is ready for review. Open a **new** reviewer session and run `/dev-nika-workflow:review NNN`." This session never reviews its own code, not even through a subagent.
