# dev-flow

A Claude Code plugin that packages a multi-role development workflow. Each role runs in its own Claude Code session:

| Command | Role | What it does |
|---|---|---|
| `/dev-flow:lead` | Team lead | The only orchestrator: plan board, work-in-progress limit, E2E slot, proposes the next session, checks reports |
| `/dev-flow:plan <topic or NNN>` | Planner | Problem statement → requirements, decision records, contracts → a plan at *Ready for implementation* |
| `/dev-flow:implement NNN [fix rK]` | Implementer | Builds one plan (typing the command is the per-plan permission), or fixes a review round |
| `/dev-flow:review NNN` | Reviewer | Independent review in a session that did not write the code; writes a review file with a verdict |
| `/dev-flow:qa NNN` | QA | Automated gates, then E2E over the plan's scenarios, until *QA passed* |

Plus a `dev-flow:code-scout` subagent (Haiku, read-only) for cheap mechanical code search.

The roles are user-invoked only (`disable-model-invocation: true`): Claude never starts a role on its own.

This repository is both the **marketplace** (`dev-flow-tools`) and the **plugin** (`dev-flow`).

## Layout

```
.claude-plugin/marketplace.json        marketplace "dev-flow-tools", one plugin entry
plugins/dev-flow/
  .claude-plugin/plugin.json           plugin manifest (name + description, no version yet)
  skills/{lead,plan,implement,review,qa}/SKILL.md
  agents/code-scout.md
  references/workflow-rules.md         the portable rules every skill reads
  references/config-interview.md       how a skill creates a missing project config
  templates/plan-template.md
  templates/review-template.md
  templates/workflow-config.template.md
README.md                              this file
ADOPT.md                               how to adopt the workflow in a project
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

The source of truth is [`references/workflow-rules.md` § 5](plugins/dev-flow/references/workflow-rules.md#5-supporting-skills-only-with-user-approval); this section only summarizes it. In short:

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

dev-flow does **not** bundle any of these skills. Checked on the computer where dev-flow was written (`claude plugin list`, Claude Code 2.1.312):

| Skills | Provided by | Confirmed install source |
|---|---|---|
| `superpowers:*` | The **Superpowers** plugin (`superpowers`) | Installed as `superpowers@synced`, i.e. synced from claude.ai (Anthropic Directory). No CLI marketplace name was shown, so no install command is given here: find it in the Anthropic Directory on claude.ai or with `/plugin` in Claude Code. |
| `design:*` | The **Design** plugin (`design`) | Installed as `design@synced`, same as above. |
| `code-review`, `security-review` | Built into Claude Code (no plugin) | Present without a namespace in the session's skill list. |

Do not confuse the built-in `code-review` with `engineering:code-review` from the Engineering plugin; dev-flow means the built-in one.

Without these plugins dev-flow still works: the roles simply propose fewer skills and do the same checks by hand.

### Not used with dev-flow

These conflict with plan files, role separation or dev-flow's git rules, so the roles never propose them: `superpowers:brainstorming`, `superpowers:writing-plans`, `superpowers:executing-plans`, `superpowers:subagent-driven-development`, `superpowers:using-git-worktrees`, `superpowers:finishing-a-development-branch`, `superpowers:requesting-code-review`. If another plugin tells Claude to run a skill automatically (for example a SessionStart message), dev-flow's approval rule wins.

## Install locally (this computer)

```bash
claude plugin marketplace add "C:\Zulfan Ahmadi\Code Project\dev-flow"
```

```bash
claude plugin install dev-flow@dev-flow-tools
```

Restart Claude Code (or start a new session) and type `/dev-flow:` to see the commands.

`plugin.json` has no `version` field on purpose. With a version set, installs stay on the cached copy until the version is bumped. After editing the plugin, refresh the installed copy and start a new session:

```bash
claude plugin marketplace update dev-flow-tools
```

```bash
claude plugin update dev-flow@dev-flow-tools
```

## Test while editing

Load the plugin straight from the folder for one session, without installing it:

```bash
claude --plugin-dir "C:\Zulfan Ahmadi\Code Project\dev-flow\plugins\dev-flow"
```

Validate the manifests, skills and agent:

```bash
claude plugin validate "C:\Zulfan Ahmadi\Code Project\dev-flow\plugins\dev-flow"
```

```bash
claude plugin validate "C:\Zulfan Ahmadi\Code Project\dev-flow"
```

Add `--strict` to treat warnings as errors.

## Install from GitHub (other computers, teammates)

The repository is published privately at [ZulfanAhmadi12/dev-Claude-Workflow](https://github.com/ZulfanAhmadi12/dev-Claude-Workflow). The repository name differs from the plugin name: the plugin stays `dev-flow` and the marketplace stays `dev-flow-tools`, so the commands stay `/dev-flow:*`.

1. **Get git access first.** The repository is private, so the computer must be able to clone it before Claude Code can add it. Sign in with the GitHub CLI (or set up an SSH key or git credential helper that has access):

   ```bash
   gh auth login
   ```

   Check it with `git ls-remote https://github.com/ZulfanAhmadi12/dev-Claude-Workflow.git`. A teammate also needs to be added as a collaborator on the repository.
2. **Add the marketplace and install the plugin.** Inside Claude Code:

   ```
   /plugin marketplace add ZulfanAhmadi12/dev-Claude-Workflow
   /plugin install dev-flow@dev-flow-tools
   ```

   Or from a terminal:

   ```bash
   claude plugin marketplace add ZulfanAhmadi12/dev-Claude-Workflow
   ```

   ```bash
   claude plugin install dev-flow@dev-flow-tools
   ```

3. **On a computer that already added the local folder**, remove the local marketplace first, because both use the name `dev-flow-tools`:

   ```bash
   claude plugin marketplace remove dev-flow-tools
   ```

   Then run step 2.
4. **Publishing changes:** commit in this repository and push to `main`; other computers pick it up with `claude plugin marketplace update dev-flow-tools` and `claude plugin update dev-flow@dev-flow-tools`.
5. Once other people use it, add `"version": "0.1.0"` to `plugins/dev-flow/.claude-plugin/plugin.json` and bump it on every release, so their installs update deliberately rather than on every commit.
6. To have a project offer the plugin to teammates automatically, declare the marketplace and plugin in that project's `.claude/settings.json` (`extraKnownMarketplaces` and `enabledPlugins`); see ADOPT.md step 6.

## Where the rules came from

The workflow was first built inside one project and then generalized. Project-specific rules (crypto rules, ports, supported browsers, reset commands, package manager, docs language, design system) moved into the per-project config. Portable rules stayed in `references/workflow-rules.md`, together with lessons learned from running it: every report carries the branch tip SHA and the `git merge-base --is-ancestor` result, QA commits partial results at every pause, plans have an Edge cases line, QA batches all user-only steps into one `NEEDS USER INPUT` message, and the config has a demo-setup runbook.
