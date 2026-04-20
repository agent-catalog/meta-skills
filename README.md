# agent-catalog meta-skills

Four SKILL.md skills for creating, migrating, validating, and deploying agent-catalog manifests. Each skill works on its own — install only what you need.

## Skills

**`agent-catalog-author`**

Start from nothing. Walks through each of the seven entry types interactively, asks what the publisher has, applies best-practices description style, and outputs a YAML source file ready for validation.

**`agent-catalog-migrate`**

Already have standard well-known files on your origin? This skill fetches `/.well-known/api-catalog`, `/.well-known/oauth-authorization-server`, `/.well-known/openid-configuration`, `/.well-known/mcp.json`, `/.well-known/agent-card.json`, `/.well-known/web-bot-auth`, `/llms.txt`, `/AGENTS.md`, `/sitemap.xml`, and `/security.txt`, then assembles a draft catalog that aggregates whatever it finds and flags what's missing.

**`agent-catalog-validate`**

Wraps `npx agent-catalog verify` with model-friendly error output. Validates the catalog against the JSON Schema, checks that all `requires` cross-references resolve, fetches referenced URLs and verifies SHA-256 hashes, and verifies the Sigstore signature if present. Reports pass/fail per entry.

**`agent-catalog-publish`**

Wraps `npx agent-catalog build` and helps the publisher choose a deployment target — static site, S3/R2 bucket, or a well-known route on an existing Node/Python/Go server. Produces deploy-ready artifacts and a deployment walkthrough customized for the chosen target.

## Workflow

```
agent-catalog-author     ← zero → draft
        OR
agent-catalog-migrate    ← existing standards → draft
        ↓
agent-catalog-validate   ← draft → verified
        ↓
agent-catalog-publish    ← verified → deployed
```

## Install

Via the vercel-labs/skills CLI:

```bash
# all four
npx skills add vercel-labs/agent-skills --skill 'agent-catalog-*'

# individually
npx skills add vercel-labs/agent-skills --skill agent-catalog-author
npx skills add vercel-labs/agent-skills --skill agent-catalog-migrate
npx skills add vercel-labs/agent-skills --skill agent-catalog-validate
npx skills add vercel-labs/agent-skills --skill agent-catalog-publish
```

The CLI detects your agent harness (Claude Code, Cursor, Codex, Cline, Continue, Goose, Windsurf, etc.) and installs to the right directory.

## Related repos

- [agent-catalog/spec](https://github.com/agent-catalog/spec) — the normative specification and JSON Schema
- [agent-catalog/server](https://github.com/agent-catalog/server) — reference server and CLI
- [agent-catalog/examples](https://github.com/agent-catalog/examples) — gold-standard example deployment
