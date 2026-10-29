# Workflow config — <project name>

Project config for the dev-flow plugin. Every dev-flow skill reads this file first. Keep the section headings as they are; the skills look them up by name. Write "none" for a section that does not apply instead of deleting it.

## Project

- Name: <project name>
- One-line description: <what the product is and who uses it>
- Main checkout path: <absolute path of the main checkout>
- Instruction files sessions must also read: <e.g. AGENTS.md, CLAUDE.md, client/AGENTS.md, or none>

## Languages

- Docs (plans, reviews, decision records, status files): <e.g. English | Indonesian>
- Chat with the user: <e.g. English>
- User-facing UI copy: <e.g. Indonesian, or n/a>
- Technical identifiers (tables, columns, types, endpoints): <e.g. English>
- Glossary of locked domain terms: <path or none>

## Branches

- Integration branch: <e.g. develop>
- Main branch: <e.g. main>
- Worktree dir: <e.g. .claude/worktrees/>
- Branch pattern for implementer: <e.g. claude/plan-NNN-slug>
- Branch pattern for reviewer: <e.g. claude/review-NNN-rK>
- Docs-only paths (a branch touching only these may be fast-forwarded by a session): <e.g. docs/**, AGENTS.md, CLAUDE.md>
- Build/merge notes: <anything special, e.g. "Turbopack hangs in fresh worktrees; use `next build --webpack`", or none>

## Docs locations

Write "none" for any the project does not have; the skills skip the matching step.

- Plans dir: <e.g. docs/plans/>
- Reviews dir: <e.g. docs/reviews/>
- Plan template: <plugin default, or a project path such as docs/plans/README.md>
- Review template: <plugin default, or a project path>
- Decision record (ADR) index: <e.g. docs/status/architecture-decisions.md>
- Decision record dir and naming: <e.g. docs/status/adr/ADR-NNN-slug.md>
- Requirements (PRD) location: <e.g. docs/product/>
- Contract files the implementer must not edit: <e.g. docs/engineering/API_CONTRACT.md, openapi.yaml, DATABASE_MODEL.md>
- Feature catalog: <e.g. docs/status/feature-catalog.md>
- Current status: <e.g. docs/status/current-status.md>
- Lessons learned: <e.g. docs/status/LESSONS_LEARNED.md>
- Other docs the implementer may update: <e.g. docs/user-manual/**, or none>
- Docs map / reading routes: <e.g. docs/README.md, or none>

## Converted documents

Where Markdown conversions of non-text files (PDF, DOCX, XLSX, PPTX) are stored, and which converter this project uses (workflow rules § 9).

- Storage: <e.g. next to the original, same name with .md | docs/converted/>
- Converter: <e.g. markitdown; pandoc for DOCX; pdftotext for text PDFs>
- Install note: <e.g. "uv tool install markitdown", or none>

## Risk tier A

A plan is tier A if it touches any of these. Everything else is tier B (or C for typo/one-file/docs fixes).

- <e.g. authentication and sessions>
- <e.g. authorization: roles, permissions, record scope>
- <e.g. cryptography, keys, encrypted fields>
- <e.g. database schema and migrations>
- <e.g. public API contract>
- <e.g. secret files, payment, personal data>

## Models

| Tier | Planner | Implementer | Reviewer |
|---|---|---|---|
| A | <e.g. Opus> | <e.g. Opus> | <e.g. Opus, with security-review + code-review> |
| B | <e.g. Opus> | <e.g. Sonnet; Opus for large or integration-heavy plans> | <e.g. Opus, short review> |
| C | — | <e.g. Sonnet> | <not required> |

## Commands

Run from: <repo root, or a sub-folder>
Package manager: <e.g. bun | npm | pnpm | uv | cargo> — <any rule, e.g. "never use npm">

- Install: <command>
- Dev server(s): <command and ports>
- Lint: <command>
- Type-check: <command or n/a>
- Test (all): <command>
- Test (focused file): <example command>
- Build: <command; when it is required, e.g. "for build/config/routing changes">
- Other gates: <e.g. test-vector verification, or none>

Gate order for implementer and QA: <e.g. lint → type-check → test → build>

## Test isolation

- Test database rule: <e.g. "tests force DATABASE_URL to myapp_test; never run tests in a way that skips tests/setup-env.ts" or n/a>
- Per-plan test database for implementer: <e.g. TEST_DATABASE_NAME=myapp_test_p<NNN>>
- Per-review test database for reviewer: <e.g. TEST_DATABASE_NAME=myapp_test_review_<NNN>>
- Other test conventions sessions must know: <e.g. "DOM tests must be .tsx", or none>

## E2E environment

- Shared dev environment the E2E slot protects: <e.g. the dev database myapp_dev>
- Reset command (the only reset sessions may run, and only while holding the E2E slot): <e.g. bun run db:dev:e2e:reset, or none>
- Run mode for E2E: <e.g. "set USE_MOCK=false, restart the dev server">
- Where the E2E slot queue is recorded: <e.g. Current status file, section "E2E slot">
- Test data rule: <e.g. "synthetic data only; never read real employee data">

## Browser and manual QA

- Browser for E2E: <e.g. real Chrome via Claude in Chrome | built-in browser pane | Playwright>
- Known browser limits: <e.g. "the desktop app's browser pane cannot load SharedWorker; use real Chrome", or none>
- Steps that always need the user (mark "Needs manual verification", never "Passed"): <e.g. typing passphrases, SSO login, hardware keys, payment>
- Where the shared Needs manual verification list lives: <e.g. Current status file, section "Needs manual verification">

## Demo setup runbook

How to bring the app to a known demo/E2E state, step by step, so QA and the user's manual sessions start from the same place.

1. <e.g. reset command>
2. <e.g. start dev servers with the API mode flag>
3. <e.g. log in as dev user X and create the sample records Y>
4. <accounts and roles to use for each QA role variant>

## Non-negotiable rules

Project rules that every role must follow and the reviewer must check. Each line is a rule a finding can cite.

1. <e.g. the server never decrypts field X>
2. <e.g. no secrets, personal data or plaintext of field X in logs, URLs, errors>
3. <e.g. only browsers X and Y are supported>

## Forbidden actions

Actions no session may take in this project, in addition to the plugin's rules (no push, no merge of code branches, no implementation without per-plan permission).

- <e.g. never read crypto/secret/ or any .env except .env.example>
- <e.g. never add a service, ORM or dependency without a decision record>
- <e.g. never edit test fixtures or skip tests to make gates pass>
- <e.g. never reset or delete data except the E2E reset above>

## Secret paths

Paths no session or subagent may read or quote (it may say a file exists, never its content). The code-scout subagent reads this list.

- <e.g. .env (all except .env.example)>
- <e.g. secrets/, infrastructure/seed/secret/>

## Area-specific reading

What to read before touching an area. Leave rows that do not apply as "none".

| Area | Read first |
|---|---|
| UI | <e.g. design.md §2, §19–21 plus the sections mapped in docs/README.md> |
| Security-sensitive (tier A) | <e.g. the crypto spec sections and test vectors> |
| Database | <e.g. lessons learned "Migrations" section> |

## Review checklist additions

Project-specific checks the reviewer adds to its manual pass.

- <e.g. route guard order: auth → role → record scope (404, not 403) → handler → audit log>
- <e.g. UI: loading/empty/error/disabled states, keyboard focus, light/dark, copy language>

## Supporting skills

- Extra skills to propose: <name — role/step — when, or none>
- Skills never to use in this project: <names, or none>

## External implementer

- Other agent that may hold the implementer role when the user assigns a plan: <e.g. Codex, branch pattern codex/plan-NNN-*, worktree path; or none>
