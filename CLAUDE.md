# CLAUDE.md — doku-codegen Plugin

This file provides guidance to Claude Code when working with the doku-codegen plugin itself.

## What this plugin does

doku-codegen generates production-quality DOKU payment API client code for any language. It auto-detects the project tech stack, fetches live API specs from developers.doku.com, and generates idiomatic code with correct DOKU signature implementations.

## Architecture

```
agents/                     ← Specialized subagents (dispatched by skills)
  sdk-generator.md          ← Opus; writes all code files
  integration-validator.md  ← Sonnet; audits generated files for security/correctness

skills/                     ← User-facing skill entry points
  setup-project/            ← Main orchestrator: detect → fetch spec → generate → validate
  fetch-api-spec/           ← Fetch live spec from developers.doku.com via sitemap
  detect-stack/             ← Detect language/framework
  setup-credentials/        ← Collect and store CLIENT_ID, SECRET_KEY
  mock-test/                ← Test against DOKU sandbox
  production-checklist/     ← Pre-go-live readiness checks
  upgrade/                  ← Diff old/new spec, patch only changed files
  generate-postman/         ← Generate Postman collection with signature script

commands/                   ← Short /slash command aliases for skills
  generate.md               ← /doku-codegen:generate → setup-project
  spec.md                   ← /doku-codegen:spec    → fetch-api-spec
  test.md                   ← /doku-codegen:test    → mock-test
  checklist.md              ← /doku-codegen:production-checklist → production-checklist
  postman.md                ← /doku-codegen:generate-postman → generate-postman
  save-session.md           ← /doku-codegen:save-session → save generation progress
  resume-session.md         ← /doku-codegen:resume-session → restore saved session

hooks/hooks.json            ← Auto-loaded event hooks (13 hooks across 5 event types)
scripts/hooks/              ← Node.js hook scripts (run via run-with-flags.js)
scripts/lib/hook-flags.js   ← DOKU_HOOK_PROFILE / DOKU_DISABLED_HOOKS gating
tests/run-all.js            ← Hook script unit tests (20 tests)

standards/                  ← Security + coding standards read by sdk-generator
  security.md, coding.md, java.md, kotlin.md, nodejs.md, python.md, php.md, golang.md
```

## Running tests

```bash
node tests/run-all.js    # runs all 19 hook script unit tests
```

Run after modifying any script in `scripts/hooks/`.

## Key commands

- `/doku-codegen:generate [payment-method]` — generate client code (main entry point)
- `/doku-codegen:spec [payment-method]` — fetch/refresh API spec
- `/doku-codegen:test` — test against sandbox
- `/doku-codegen:production-checklist` — production readiness
- `/doku-codegen:generate-postman` — generate Postman collection
- `/doku-codegen:save-session` — save progress to resume later
- `/doku-codegen:resume-session` — restore a saved session after context reset

## Hook controls

```bash
export DOKU_HOOK_PROFILE=minimal    # suppress non-critical hooks
export DOKU_DISABLED_HOOKS=post-write-java-check   # disable specific hook
```

## How agents and skills work together

Skills are orchestrators — they gather context, ask questions, confirm with users, then dispatch to agents.
Agents do the actual work: sdk-generator writes files, integration-validator audits them.
Skills never write code directly.

## When modifying this plugin

- Agents: YAML frontmatter must have `name`, `description`, `tools`, `model`
- Skills: use `name`, `description`, `origin: doku-codegen` in frontmatter
- Commands: must have `description:` frontmatter
- `plugin.json`: agents require explicit file paths; skills/commands can use directories
- Never declare hooks in `plugin.json` — `hooks/hooks.json` is auto-loaded
- Always test hook scripts: `node scripts/hooks/[script].js` with sample stdin

## DOKU-specific knowledge

- Non-SNAP signature: HMAC-SHA256, `Request-Target` uses `template.path()` (NEVER hardcoded)
- SNAP signature: HMAC-SHA512 with B2B token from RSA-signed token endpoint
- developers.doku.com is JS-rendered (GitBook) — always use `sitemap-pages.xml` for URL discovery
- Known doc error: Alfa Group DGPC endpoint in docs is wrong. Actual: `/alfa-online-to-offline/v2/payment-code`
