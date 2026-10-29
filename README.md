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

## Move to a private GitHub repository later

1. Create an empty **private** repository on GitHub, for example `<owner>/dev-flow`.
2. Push this repository to it:

   ```bash
   git -C "C:\Zulfan Ahmadi\Code Project\dev-flow" remote add origin git@github.com:<owner>/dev-flow.git
   ```

   ```bash
   git -C "C:\Zulfan Ahmadi\Code Project\dev-flow" push -u origin main
   ```

3. On each computer, replace the local marketplace with the GitHub one (the marketplace name stays `dev-flow-tools`):

   ```bash
   claude plugin marketplace remove dev-flow-tools
   ```

   ```bash
   claude plugin marketplace add <owner>/dev-flow
   ```

   ```bash
   claude plugin install dev-flow@dev-flow-tools
   ```

   A private repository needs git access on that computer (an SSH key or a credential helper that can clone it).
4. Once other people use it, add `"version": "0.1.0"` to `plugins/dev-flow/.claude-plugin/plugin.json` and bump it on every release, so their installs update deliberately rather than on every commit.
5. To have a project offer the plugin to teammates automatically, declare the marketplace and plugin in that project's `.claude/settings.json` (`extraKnownMarketplaces` and `enabledPlugins`); see ADOPT.md step 6.

## Where the rules came from

The workflow was first built inside one project and then generalized. Project-specific rules (crypto rules, ports, supported browsers, reset commands, package manager, docs language, design system) moved into the per-project config. Portable rules stayed in `references/workflow-rules.md`, together with lessons learned from running it: every report carries the branch tip SHA and the `git merge-base --is-ancestor` result, QA commits partial results at every pause, plans have an Edge cases line, QA batches all user-only steps into one `NEEDS USER INPUT` message, and the config has a demo-setup runbook.
