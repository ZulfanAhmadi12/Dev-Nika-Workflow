# Adopting dev-flow in a project

Do these steps once per project. Install the plugin first (README.md, "Install locally" on this computer, or "Install from GitHub" elsewhere).

## 1. Make sure the branches exist

The workflow expects an **integration branch** (where finished plans land, for example `develop`) and a **main branch** (releases, for example `main`). If the project only has `main`, either create `develop` from it or set both to `main` in the config.

```bash
git switch -c develop
```

## 2. Create the config

Open Claude Code in the project and run any role, for example:

```
/dev-flow:lead status
```

The skill sees that `.claude/workflow.md` is missing, stops, and offers to create it. It reads the repository first (branches, package scripts, existing docs, test setup) and then asks you only about what it could not find. Expect questions on:

- what counts as **risk tier A** here (auth, permissions, payments, schema, encryption, public API…);
- lint, type-check, test and build commands;
- how tests are isolated from dev data, and whether there is a shared dev environment that needs an **E2E slot** and a reset command;
- how browser and manual QA works, and which steps only you can do;
- integration and main branch, worktree folder, branch naming;
- where plans, reviews, decision records and status files live;
- docs language and chat language;
- forbidden actions, non-negotiable rules and secret paths;
- a **demo setup runbook**: the steps that bring the app to a known state for QA.

You can also copy the template and fill it in by hand: it is `plugins/dev-flow/templates/workflow-config.template.md` in this repository. Keep every heading; write "none" where a section does not apply.

## 3. Commit the config to the integration branch

Implementer and reviewer sessions work in git worktrees made from the integration branch. An uncommitted config exists only in the main checkout, so those sessions would not see it.

```bash
git add -- .claude/workflow.md
```

```bash
git commit -m "chore: add dev-flow workflow config" -- .claude/workflow.md
```

## 4. Create the docs folders

The default layout is `docs/plans/` and `docs/reviews/`, each with a short `README.md` that points at the plugin templates (the config interview offers to create them). Optional shared status files: a feature catalog, a current-status file (summary, E2E slot queue, Needs manual verification list) and a lessons-learned log. A project without them sets those entries to "none".

## 5. Point always-loaded instructions at the workflow

The plugin's own files load only when a skill runs. So that every session, including ones that never type a skill command, knows the basic rules, add a short section to the project's `CLAUDE.md` (or `AGENTS.md`):

```markdown
## Workflow (dev-flow plugin)

This project uses the dev-flow plugin; its config is `.claude/workflow.md`.
- Roles run in separate sessions: `/dev-flow:lead`, `/dev-flow:plan`, `/dev-flow:implement NNN`, `/dev-flow:review NNN`, `/dev-flow:qa NNN`.
- No application code, tests, config, migrations or dependencies are changed without the user naming a plan at "Ready for implementation" in the current conversation.
- Code is never reviewed by the session that wrote it.
- Sessions never push and never merge code branches; the user does.
```

## 6. Optional: share with teammates

The plugin is published at `ZulfanAhmadi12/dev-Claude-Workflow` (README.md, "Install from GitHub"). Add this to the project's `.claude/settings.json` so Claude Code offers the marketplace and plugin to everyone who opens the project:

```json
{
  "extraKnownMarketplaces": {
    "dev-flow-tools": {
      "source": { "source": "github", "repo": "ZulfanAhmadi12/dev-Claude-Workflow" }
    }
  },
  "enabledPlugins": {
    "dev-flow@dev-flow-tools": true
  }
}
```

Each teammate needs read access to the private repository and must sign in on their computer first (`gh auth login`, or git credentials / an SSH key that can clone it).

## 7. First run

1. `/dev-flow:plan <topic>` in the main checkout: agree the problem statement, get a plan to *Ready for implementation*.
2. `/dev-flow:lead dispatch NNN`: it proposes an implementer session card with the right model.
3. In the new session, `/dev-flow:implement NNN`.
4. In another new session, `/dev-flow:review NNN`.
5. Back in the implementer session, `/dev-flow:qa NNN` when the review says Go to QA and the E2E slot is free.
6. Run the merge and push commands QA gives you.

## Migrating a project that already has its own role skills

If a project already has project-level role skills (for example `.claude/skills/<project>-plan/`), the plugin's commands do not clash with them because plugin skills are namespaced (`/dev-flow:plan`). Move the project-specific parts of those skills into `.claude/workflow.md`, try the plugin on one plan, and remove the old skills once you are happy. Plans and reviews already written stay as they are; their status names and section titles may differ from the plugin template, which is fine as long as the config's "Plan template" entry points at the project's own template.
