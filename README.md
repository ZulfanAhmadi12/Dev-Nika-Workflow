# Dev-Nika-Workflow

A Claude Code plugin that packages a multi-role development workflow. Each role runs in its own Claude Code session:

| Command | Role | What it does |
|---|---|---|
| `/dev-nika-workflow:lead` | Team lead | The only orchestrator: plan board, work-in-progress limit, E2E slot, proposes the next session, checks reports |
| `/dev-nika-workflow:plan <topic or NNN>` | Planner | Problem statement → requirements, decision records, contracts → a plan at *Ready for implementation* |
| `/dev-nika-workflow:implement NNN [fix rK]` | Implementer | Builds one plan (typing the command is the per-plan permission), or fixes a review round |
| `/dev-nika-workflow:review NNN` | Reviewer | Independent review in a session that did not write the code; writes a review file with a verdict |
| `/dev-nika-workflow:qa NNN` | QA | Automated gates, then E2E over the plan's scenarios, until *QA passed* |

Plus a `dev-nika-workflow:code-scout` subagent (Haiku, read-only) for cheap mechanical code search.

The roles are user-invoked only (`disable-model-invocation: true`): Claude never starts a role on its own.

This repository is both the **marketplace** (`dev-nika-workflow-tools`) and the **plugin** (`dev-nika-workflow`).

## Layout

```
.claude-plugin/marketplace.json        marketplace "dev-nika-workflow-tools", one plugin entry
plugins/dev-nika-workflow/
  .claude-plugin/plugin.json           plugin manifest (name, displayName, version, author, license)
  skills/{lead,plan,implement,review,qa}/SKILL.md
  agents/code-scout.md
  references/workflow-rules.md         the portable rules every skill reads
  references/config-interview.md       how a skill creates a missing project config
  templates/plan-template.md
  templates/review-template.md
  templates/workflow-config.template.md
  README.md, LICENSE                   short listing README and MIT license for the plugin folder
README.md                              this file
ADOPT.md                               how to adopt the workflow in a project
LICENSE                                MIT
```

## How it is configured per project

Every skill first reads two files:

1. `references/workflow-rules.md` in the plugin: the rules that are the same everywhere (roles, per-plan permission, risk tiers, WIP limit, one E2E slot, review independence, git and merge/push rules, session report, Definition of Done).
2. `.claude/workflow.md` in the project: everything project-specific (risk tier A areas, commands, test-DB isolation, E2E reset, browser/manual QA, branches, docs locations, languages, forbidden actions, secret paths, demo-setup runbook).

If the project has no `.claude/workflow.md`, the skill stops and offers to create it by interviewing you. See [ADOPT.md](ADOPT.md).

## Reading documents

Every role follows one rule before reading a PDF, DOCX, XLSX or PPTX (`references/workflow-rules.md` § 9):

1. Referenced again → convert it once to Markdown with a CLI tool (`markitdown`, `pandoc`, `pdftotext`), store it next to the original (or where the config's **Converted documents** section says), spot-check a table, and read the Markdown from then on.
2. One-off read → search first, then read only the pages or sections needed.
3. Meaning depends on visuals (scans, charts, diagrams, layout, mockups) → read the original pages. Correctness beats cost.
4. More than ~20 pages, or large logs → a subagent reads them and returns a summary.
5. Prefer a CLI converter over adding an MCP server.

Why: you pay per token, and anything read stays in context and is re-sent on every later turn. A PDF page read directly costs its text plus a page image, roughly 2–3× its plain-text size. Converting once and reusing the Markdown is where the real saving comes from. MCP tool definitions also occupy context on every turn even when unused, which is why a CLI converter is preferred.

## Supporting skills

The source of truth is [`references/workflow-rules.md` § 5](plugins/dev-nika-workflow/references/workflow-rules.md#5-supporting-skills-only-with-user-approval); this section only summarizes it. In short:

- Each role proposes its supporting skills **once, at the start**, in one message: the skill, the step where it is used, and a reason specific to this plan.
- You approve all, some or none. **Nothing runs without approval**; a declined skill is replaced by the same checks done by hand, and is not proposed again without a new reason.
- Only skills that are **installed in the session** are proposed. A project can add or forbid skills in its config (`.claude/workflow.md` § Supporting skills).

| Role | Skill | When |
|---|---|---|
| Plan | `design:ux-copy` | The plan writes user-facing copy (labels, errors, empty states, confirmations) |
| Implement | `superpowers:test-driven-development` | Bug fixes, regression tests that must fail first, security acceptance criteria |
| Implement | `superpowers:systematic-debugging` | Mid-way, when a test fails and the root cause is unclear after one check |
| Implement | `superpowers:verification-before-completion` | Before claiming gate 2 (Implementation done) |
| Implement | `code-review` | Before hand-off, for tier B plans and tier C changes |
| Review | `code-review` | First pass for tier A (tier B: the implementer already ran it); level chosen from size and risk |
| Review | `security-review` | Tier A, or a B plan raised to A because the range touches tier A areas |
| Review | `design:accessibility-review` | The range changes screens or interactive components |
| QA | `design:accessibility-review` | UI scenarios |
| QA | `design:ux-copy` | Scenarios with user-facing copy |
| QA | `superpowers:verification-before-completion` | Before claiming QA passed |
| QA | `superpowers:systematic-debugging` | Mid-way, when a scenario fails and the root cause is unclear |

The lead proposes no supporting skills of its own; each dispatched session proposes its own.

### Where they come from

Dev-Nika-Workflow does **not** bundle any of these skills. Checked on the computer where Dev-Nika-Workflow was written (`claude plugin list`, Claude Code 2.1.312):

| Skills | Provided by | Confirmed install source |
|---|---|---|
| `superpowers:*` | The **Superpowers** plugin (`superpowers`) | Installed as `superpowers@synced`, i.e. synced from claude.ai (Anthropic Directory). No CLI marketplace name was shown, so no install command is given here: find it in the Anthropic Directory on claude.ai or with `/plugin` in Claude Code. |
| `design:*` | The **Design** plugin (`design`) | Installed as `design@synced`, same as above. |
| `code-review`, `security-review` | Built into Claude Code (no plugin) | Present without a namespace in the session's skill list. |

Do not confuse the built-in `code-review` with `engineering:code-review` from the Engineering plugin; Dev-Nika-Workflow means the built-in one.

Without these plugins Dev-Nika-Workflow still works: the roles simply propose fewer skills and do the same checks by hand.

### Not used with Dev-Nika-Workflow

These conflict with plan files, role separation or Dev-Nika-Workflow's git rules, so the roles never propose them: `superpowers:brainstorming`, `superpowers:writing-plans`, `superpowers:executing-plans`, `superpowers:subagent-driven-development`, `superpowers:using-git-worktrees`, `superpowers:finishing-a-development-branch`, `superpowers:requesting-code-review`. If another plugin tells Claude to run a skill automatically (for example a SessionStart message), Dev-Nika-Workflow's approval rule wins.

## Install

Inside Claude Code:

```
/plugin marketplace add ZulfanAhmadi12/dev-Claude-Workflow
/plugin install dev-nika-workflow@dev-nika-workflow-tools
```

Or from a terminal:

```bash
claude plugin marketplace add ZulfanAhmadi12/dev-Claude-Workflow
```

```bash
claude plugin install dev-nika-workflow@dev-nika-workflow-tools
```

Restart Claude Code (or start a new session) and type `/dev-nika-workflow:` to see the commands. Then follow [ADOPT.md](ADOPT.md) in each project.

If you added a local clone as the marketplace earlier, remove it first, because both use the name `dev-nika-workflow-tools`:

```bash
claude plugin marketplace remove dev-nika-workflow-tools
```

To have a project offer the plugin to everyone who opens it, declare the marketplace and plugin in that project's `.claude/settings.json` (`extraKnownMarketplaces` and `enabledPlugins`); see ADOPT.md step 6.

## Updates and versions

`plugins/dev-nika-workflow/.claude-plugin/plugin.json` has a `version` (currently `0.1.0`). Installed copies stay on the cached version until that number changes, so **every release must bump `version`** (semver: patch for wording fixes, minor for new rules or skills, major for changes that break existing project configs). Pushing commits without a bump does not reach users.

To pick up a new release:

```bash
claude plugin marketplace update dev-nika-workflow-tools
```

```bash
claude plugin update dev-nika-workflow@dev-nika-workflow-tools
```

## Develop locally

Clone the repository, then load the plugin straight from the folder for one session, without installing it:

```bash
claude --plugin-dir "<path-to-clone>/plugins/dev-nika-workflow"
```

Or register the clone as the marketplace (instead of the GitHub one) and install from it:

```bash
claude plugin marketplace add "<path-to-clone>"
```

Validate the plugin and the marketplace before every release:

```bash
claude plugin validate "<path-to-clone>/plugins/dev-nika-workflow"
```

```bash
claude plugin validate "<path-to-clone>"
```

Add `--strict` to treat warnings as errors.

## Where the rules came from

The workflow was first built inside one project and then generalized. Project-specific rules (crypto rules, ports, supported browsers, reset commands, package manager, docs language, design system) moved into the per-project config. Portable rules stayed in `references/workflow-rules.md`, together with lessons learned from running it: every report carries the branch tip SHA and the `git merge-base --is-ancestor` result, QA commits partial results at every pause, plans have an Edge cases line, QA batches all user-only steps into one `NEEDS USER INPUT` message, and the config has a demo-setup runbook.
