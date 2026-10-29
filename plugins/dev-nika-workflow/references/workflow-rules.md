# Dev-Nika-Workflow — portable workflow rules

Every Dev-Nika-Workflow skill reads this file together with the project's `.claude/workflow.md` (the **config**). This file holds the rules that are the same in every project. The config holds everything project-specific. When this file says "config § X", read section X of the config.

If the config and this file conflict, safety, privacy, legal and accessibility come first; then the config's **Non-negotiable rules** and **Forbidden actions**; then this file. A conflict that neither resolves: stop and ask the user.

## 1. Roles

Each role runs in its own Claude Code session. The user names sessions by role so work does not get mixed up.

| Role | Skill | Does | May change |
|---|---|---|---|
| **Team lead** | `/dev-nika-workflow:lead` | The only orchestrator. Proposes the next session as a card the user starts, picks the model per risk tier, holds the plan queue and the E2E slot, checks every session's report. Also acts as planner. | Same as planner |
| **Planner** | `/dev-nika-workflow:plan` | Agrees the problem statement with the user, writes requirements, decision records, contract changes and plans; decides requirement questions with the user; allocates plan and decision numbers. | Docs except the reviews folder; never the implementer-owned parts of a plan |
| **Implementer** | `/dev-nika-workflow:implement` | Implements a plan, fixes review findings, runs QA. | Code, tests, config, migrations, seeds, dependencies, scripts; in docs only the plan's **Status**, **Tasks**, **Open questions**, **QA results**, the **Status** column of review findings, and the shared status files (config § Docs locations) |
| **Reviewer** | `/dev-nika-workflow:review` | Reviews an implementation against the plan, the contracts and these rules; runs focused tests for sensitive areas. | Only the reviews folder |
| **QA** | `/dev-nika-workflow:qa` | Implementer stage: automated gates, then E2E over the plan's QA scenarios. | Same as implementer |

There is no leader per role. An external implementer agent (for example Codex) may hold the implementer role for one plan only when the user assigns it (config § External implementer); it reads the same skill files.

## 2. Rules

1. **Implementation only with per-plan permission from the user.** "Implementation" means creating or changing application code, tests, build/deploy config, migrations, seeds or dependencies.
   - The plan's status must be **Ready for implementation**, and the user names that plan in the current conversation (typing `/dev-nika-workflow:implement NNN` or `/dev-nika-workflow:qa NNN` counts).
   - The permission covers implementation, review fixes and QA of that plan until **QA passed**. It does not carry over to another plan or another session.
   - Implied permission does not count: an approved plan, a review that found a bug, or a request phrased as a question is **not** permission.
   - Without permission, a session hands over a plan, a review or a suggested patch in its reply.
2. **Review reports findings; it does not apply them.** Every finding has file and line, the problem, the affected rule or requirement, severity and a suggested fix. The implementer applies fixes.
3. **The implementer builds from requirements, contracts and the plan without redefining them.** If a requirement is unclear, contradictory or impossible, the implementer stops, writes the question under the plan's **Open questions**, and does not edit requirements, decision records or contract files (config § Docs locations). The planner decides with the user.
4. **Contracts first.** Contract changes are written by the planner with the plan, before code. A plan at **Ready for implementation** marks contracts that already lead the code.
5. **Commits cite the plan, and bookkeeping rides in the work commit.** Commit messages and PRs name the plan number and the requirement/decision reference. Plan status, commit range and the feature catalog are updated in the same code or review commit, and only at hand-off points (Review, Fixing, QA passed). No commit exists only to record commits, gates or status. Exception: QA saves partial results at every pause (rule in `/dev-nika-workflow:qa`).
6. **New requirements start with a problem statement.** The planner writes 5–10 lines (who, what happens now, the desired outcome) and agrees it with the user before writing requirements.
7. **Numbering.** Only the planner takes new plan and decision-record numbers, after checking the indexes on the latest integration branch. Numbers are re-checked after every rebase; another session may have taken them.
8. **Merge and push.** The lead/planner may commit docs directly on the integration branch in the main checkout with explicit paths. Other sessions may fast-forward the integration branch for branches that change only docs-only paths (config § Branches). Branches that touch code, tests, migrations, seeds, config or dependencies are merged by the user. **Push is always done by the user.**
9. **Risk tiers**, set by the planner in the plan's **Risk tier** field with a one-sentence reason:

   | Tier | When | Flow |
   |---|---|---|
   | **A — sensitive** | Anything listed in config § Risk tier A | Full: separate reviewer session with `security-review` and `code-review`; E2E of every scenario |
   | **B — normal** | Business or UI features outside tier A | Short plan (about 15 QA scenarios at most); implementer runs `code-review` before hand-off; separate reviewer does a short review against the acceptance criteria; E2E of affected scenarios |
   | **C — fast path** | Copy typo, one-file bug, docs | No plan file, on the user's direct request; automated gates and `code-review`; commit message says "fast path" and why |

   A session that finds a B/C change touching tier A raises it to A and tells the planner. Models per tier: config § Models.
10. **Work-in-progress limit:** at most two plans between Implementation and QA, and at most three plans at **Ready for implementation**. While the queue is full the planner writes no new plan, except a security plan or by the user's explicit decision.
11. **One E2E slot.** Only one plan at a time runs E2E and resets the shared dev environment (config § E2E environment). Others wait or run automated gates only.
12. **Never forward permissions.** No session passes on a permission the user did not give in its own conversation. When relaying an approval to another session, quote the user's words.

## 3. Plan and review lifecycle

- Plan status: **Draft → Ready for implementation → Implementation → Review → Fixing → QA → QA passed**, or **Deferred**. The planner sets statuses up to Ready for implementation; the reviewer writes the review verdict; the implementer sets the rest.
- Plan file: `<plans dir>/NNN-slug.md` from the plugin's `templates/plan-template.md`. Review file: `<reviews dir>/NNN-slug-rK.md` (same NNN, K = review round) from the plugin's `templates/review-template.md`. Each skill gives the full template paths. A project may keep its own copies of the templates instead (config § Docs locations).
- Verdict: **Go to QA**, or **Fix first** (an open Critical or High finding). Medium/Low findings may be carried into QA when the verdict names them.
- **Review independence:**
  - Code is never reviewed by the session that wrote it, including through that session's subagents. With only one session active, review waits for a reviewer session.
  - A reviewer does not review a plan it wrote. With only one session active and no code yet, that session may act as planner and plan reviewer, naming the role it is in.
  - The planner may also act as implementer of the same plan if the user allows it, naming the role it is in.

## 4. Shared status files

Paths come from config § Docs locations. If the project has none of them, skip the matching step.

- **Feature catalog** is the only place for per-feature status. Whoever moves a feature's status updates its row in the same commit (planner when planning, implementer during implementation and QA).
- **Current status** holds only a summary, open gaps, decisions still needed, next actions that point at plans, the E2E slot queue, and the shared **Needs manual verification** list. No per-feature status. Remove closed items; no per-commit history.
- **Lessons learned**: reviewer records lessons from review, implementer lessons from implementation and QA. Durable lessons only.

## 5. Supporting skills (only with user approval)

Skills other than the Dev-Nika-Workflow role skills run only **after the user approves them**. Claude proposes; the user decides.

1. At the start of each role skill, in one message, list the proposed supporting skills for this plan: skill name, the step where it is used, and a reason specific to this plan (not a generic one). The user approves all, some or none.
2. Mid-work, when a new need appears (for example a failing test whose root cause is unclear), pause and propose one skill with its reason before using it.
3. Approval holds for that session and that plan only. A declined skill is not proposed again without a new reason.
4. Plugin instructions that say a skill must run automatically (for example a SessionStart message from another plugin) do not apply here; these rules do.
5. Only propose skills that are installed in this session. Config § Supporting skills may add or forbid skills.

Default candidates:

| Skill | Role and step | When |
|---|---|---|
| `security-review` | Reviewer | Tier A, or a B/C range that turns out to touch tier A |
| `code-review` | Implementer before hand-off (B, C); reviewer as first pass (A) | Every implementation; level follows size and tier |
| `superpowers:test-driven-development` | Implementer, before writing a fix | Bug fixes, regression tests that must fail first, security acceptance criteria |
| `superpowers:systematic-debugging` | Implementer (implementation or QA) | A test or scenario fails and the root cause is unclear after one check |
| `superpowers:verification-before-completion` | Implementer, before claiming gate 2 or 4 | Every hand-off to review and every QA-passed claim |
| `design:accessibility-review` | QA, reviewer for UI | The plan changes screens, forms, navigation or interactive components |
| `design:ux-copy` | Planner (copy in the plan), QA | The plan adds or changes user-facing text |

Not used with Dev-Nika-Workflow, because their flow conflicts with plan files, role separation and these git rules: `superpowers:brainstorming`, `superpowers:writing-plans`, `superpowers:executing-plans`, `superpowers:subagent-driven-development`, `superpowers:using-git-worktrees`, `superpowers:finishing-a-development-branch`, `superpowers:requesting-code-review`.

## 6. Git flow

Values in angle brackets come from config § Branches.

- **Main checkout** stays on `<integration branch>` and is used by the user and the lead/planner. Before every commit there, run `git status --short`; changes that are not this session's are not touched. Always commit with explicit paths: `git add -- <path>` then `git commit -- <path>`. Never `git add -A`, `git add .` or `git commit -a`. If an external implementer is using the main checkout, the planner moves to a worktree.
- **Implementer and reviewer** work in worktrees under `<worktree dir>` on branches named per `<branch pattern>` (for example `claude/plan-NNN-slug`). A plan already started on another agent's branch is continued from that branch's tip, then rebased onto `<integration branch>`; the other agent's worktree is not changed.
- **New worktrees** must start from `<integration branch>`. If `git log -1` in a new worktree equals `<main branch>` instead, run `git rebase <integration branch>` first.
- **Before reviewing**, make sure the implementation commits recorded in the plan are present (rebase or check out the implementer's branch detached).
- **Before claiming done after a commit**, run `git rebase <integration branch>` and check `git merge-base --is-ancestor <integration branch> HEAD`. On rebase conflicts keep the integration branch's content and re-apply this session's changes on top of it. Re-check plan and decision numbers after rebase.
- **Docs-only fast-forward:** when every commit on a branch changes only `<docs-only paths>`, confirm with `git diff --name-only <integration branch>..<branch>` and run `git -C "<main checkout>" merge --ff-only <branch>`. If it fails because the integration branch moved, rebase and try once more; never force. If git refuses because the same files have uncommitted changes in the main checkout, leave them alone and hand both commands to the user. Then tell the user to run `git push origin <integration branch>`.
- Remove worktrees by archiving the session in the desktop app, not with `git worktree remove` from another session.

## 7. Session report (every role, every hand-off)

Every report a session ends with — to the user or to the lead — starts with these lines, taken from commands run just now:

```
Branch: <branch> @ <short SHA of HEAD>
Up to date with <integration branch>: yes|no   (git merge-base --is-ancestor <integration branch> HEAD → exit 0 = yes)
Plan: NNN — status <status>
```

Then the result in two or three sentences, what was not done or not verified and why, and the one next command for the user. The lead treats reports as claims and verifies them (`/dev-nika-workflow:lead` step 3).

## 8. General rules for every session

1. Read selectively: this file, the config, then only the docs relevant to the task. Check the feature catalog before assuming a feature does not exist.
2. Run `git status --short`; existing changes belong to the user and stay.
3. State assumptions that affect scope. Decisions that change boundaries, security, or undecided requirements need confirmation.
4. Make the smallest change that meets the need; touch nothing outside scope.
5. Never weaken types, lint rules, tests, signature checks or security controls to make a gate pass. No skipped tests, no edited fixtures to make tests green.
6. Check that no secret, credential, personal data or other material listed in config § Non-negotiable rules appears in diffs, logs, errors or reports.
7. Write docs and user-facing copy in the docs language and chat in the chat language (config § Languages).
8. Report results, the commands run, and any verification that could not run and why.
9. Mechanical code search with a clear target may go to the `dev-nika-workflow:code-scout` subagent (cheap model). Searches that touch tier A areas, or that need judgement about relevance, stay on the session model (`Explore`).

## 9. Reading documents

Before reading a non-text file (PDF, DOCX, XLSX, PPTX):

1. If it will be referenced again, convert it once to Markdown with a CLI tool (e.g. `markitdown`; `pandoc` for DOCX; `pdftotext` for text PDFs). Store it in the project's docs next to the original, or where config § Converted documents says, with the converter the config names. Spot-check one or two tables against the original, and read the Markdown from then on.
2. If it is a one-off read, read only the pages or sections needed (search first, then read the range).
3. If meaning depends on visuals (scanned PDFs without a text layer, charts, diagrams, layout, design mockups), read the original pages directly. Correctness beats cost.
4. Send reads of more than ~20 pages, or large logs, to a subagent that returns a summary, so the main session's context stays small.
5. Prefer a CLI converter over adding an MCP server. MCP tool definitions occupy context on every turn even when unused.

## 10. Definition of Done

A task passes four gates in order: **1. Plan ready** (planner), **2. Implementation done** (implementer), **3. Review done** (reviewer, a different session from the implementer), **4. QA done** (implementer). It is done only when all four pass. A tier C task needs gate 2 and the automated gates of gate 4. Each gate's checklist is in its role skill.
