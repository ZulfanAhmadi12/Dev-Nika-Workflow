# Dev-Nika-Workflow

Dev-Nika-Workflow runs software work through separate Claude Code sessions with clear roles: a team lead coordinates, a planner writes plans and contracts, an implementer builds one plan at a time with the user's per-plan permission, an independent reviewer reviews code it did not write, and QA verifies with automated gates and end-to-end scenarios. Plans carry a risk tier that sets how deep review and QA go.

## Install

```
/plugin marketplace add ZulfanAhmadi12/dev-Claude-Workflow
/plugin install dev-nika-workflow@dev-nika-workflow-tools
```

## Commands

| Command | Role |
|---|---|
| `/dev-nika-workflow:lead [status \| dispatch NNN \| manual-verification]` | Team lead: plan board, work-in-progress limit, E2E slot, proposes the next session |
| `/dev-nika-workflow:plan <topic or NNN>` | Planner: problem statement, requirements, contracts, plan |
| `/dev-nika-workflow:implement NNN [fix rK]` | Implementer: builds one plan or fixes a review round |
| `/dev-nika-workflow:review NNN` | Reviewer: independent review with a verdict |
| `/dev-nika-workflow:qa NNN` | QA: gates, then E2E, until QA passed |

All five are user-invoked only; Claude never starts a role by itself. A read-only `code-scout` subagent (Haiku) handles cheap mechanical code searches.

## Setup

Each project needs a `.claude/workflow.md` config (risk tier A areas, commands, branches, docs locations, languages, forbidden actions, secret paths, E2E and manual-QA rules). If it is missing, the first role you run offers to create it by interviewing you. The template is `templates/workflow-config.template.md`.

## What it runs and sends

The plugin contains only Markdown: skills, one agent, rules and templates. It has no hooks, no MCP servers, no scripts and no network calls of its own. Its skills instruct Claude to read project files, run `git` commands and the project's own lint/test/build commands named in the config, and write plan, review and status documents in the project. They never push, never merge code branches and never send data anywhere; the user does those steps.

Full documentation, install steps and the adoption guide: <https://github.com/ZulfanAhmadi12/dev-Claude-Workflow>. License: MIT.
