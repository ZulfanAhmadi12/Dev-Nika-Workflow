# Creating `.claude/workflow.md` (config interview)

Used by every dev-flow skill when the project has no `.claude/workflow.md`. The skill that sent you here gives you the path of the config template.

1. **Stop the role work.** Tell the user in one or two lines that this project has no dev-flow config yet, and offer to create it now by interviewing them. Do nothing else until they agree. If they decline, end the turn; the role skill cannot run without the config.
2. **Gather facts before asking.** Read-only, so the user only answers what the repo cannot tell you:
   - `git branch -a`, `git remote -v`, `git log --oneline -5` (integration and main branch candidates);
   - the package manifest(s) and scripts (`package.json`, `pyproject.toml`, `Makefile`, `Cargo.toml`, …) for lint, type-check, test, build and dev commands;
   - existing instruction files (`AGENTS.md`, `CLAUDE.md`), a docs folder, an ADR index, a plans folder;
   - test setup files that hint at test-database isolation;
   - `.gitignore` and folder names that look secret (`secret`, `.env*`, `keys`). Do not open them.
3. **Draft, then ask.** Fill the template with what you found, marking every guess "(guess)". Ask the user only about the gaps and guesses, in at most two rounds, grouping questions by section. Always ask explicitly about: what counts as **Risk tier A**, the **Forbidden actions**, the **Non-negotiable rules**, the **Secret paths**, the docs and chat **Languages**, and whether there is a shared dev environment that needs an **E2E slot** and a reset command.
4. **Write** `.claude/workflow.md` in the project. Keep every section heading from the template; write "none" where a section does not apply. Remove "(guess)" only for facts the user confirmed.
5. **Docs scaffolding (optional, ask first).** If the plans or reviews dir does not exist, offer to create it with a `README.md` that points at the plugin templates. Do not create status files the user did not ask for.
6. **Commit only with the user's yes.** Propose `git add -- .claude/workflow.md` (plus any scaffolding) and `git commit -- <paths>` with a message like "chore: add dev-flow workflow config". Never push.
7. Then resume the role skill from its step 1.
