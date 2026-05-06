# doku-codegen

Generate DOKU API client code in any language — auto-detects your tech stack and fetches live API specs from developers.doku.com.

## Skills

| Skill | Trigger | Purpose |
|---|---|---|
| `detect-stack` | "detect stack" | Detect project language/framework, save to config |
| `fetch-api-spec` | "fetch DOKU API spec" | Navigate developers.doku.com, extract spec, save to config |
| `setup-project` | "generate DOKU code" | **Main entry point** — runs all prerequisites inline, generates client code |
| `setup-credentials` | "set up DOKU credentials" | Collect CLIENT_ID, SECRET_KEY, environment |
| `mock-test` | "test DOKU integration" | Send test request to sandbox, verify signature |
| `production-checklist` | "production checklist" | Run 8 readiness checks before go-live |
| `upgrade` | "upgrade DOKU client" | Diff old vs new spec, patch only changed files |
| `generate-postman` | "generate Postman collection" | Export Postman collection with signature pre-request script |

## Quick Start

```
"generate DOKU code"               ← handles everything automatically
/doku-codegen:mock-test            ← verify sandbox connection
/doku-codegen:production-checklist ← before going live
```

## Advanced Usage

```
/doku-codegen:detect-stack         ← re-run if changing language/framework
/doku-codegen:setup-credentials    ← re-run when rotating keys
/doku-codegen:fetch-api-spec       ← re-run when DOKU updates their API docs
/doku-codegen:upgrade              ← patch changed files after spec refresh
/doku-codegen:generate-postman     ← generate Postman collection at any time
```

## Supported Languages & Frameworks

| Language | Framework | Signature | Config |
|---|---|---|---|
| Java | Spring Boot 3 + Feign Client | HMAC-SHA256 / HMAC-SHA512 | application.yml |
| Python | FastAPI + httpx | HMAC-SHA256 / HMAC-SHA512 | .env + config.py |
| Node.js | Express + axios | HMAC-SHA256 / HMAC-SHA512 | .env + config.js |

## Config File

Credentials and spec are stored in `.claude/doku-codegen.local.md` (gitignored automatically).

## Design Principles

- **No hardcoded URLs** — always navigates developers.doku.com from root by keyword matching
- **Prerequisite following** — fetches auth/token pages referenced by the main API page
- **Single entry point** — `setup-project` runs missing steps inline automatically
- **Spec versioning** — `API_SPEC_PREVIOUS` archived on every refresh for diffing
- **Responsibility-based skills** — adding a new DOKU API never requires a new skill
