# agent-catalog meta-skills

Four SKILL.md skills for creating, migrating, validating, and deploying agent-catalog manifests.

## Skills

| Skill | Purpose |
|---|---|
| `agent-catalog-author` | Start from nothing. Walks through each entry type and produces a YAML source file. |
| `agent-catalog-migrate` | Already have well-known files on your origin? This scans for them and assembles a draft catalog. |
| `agent-catalog-validate` | Check a draft against the schema, verify cross-references, get readable error output. |
| `agent-catalog-publish` | Pick a deployment target and produce deploy-ready artifacts. |

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

Each skill works on its own. If you already have a draft, skip to validate. Already validated? Skip to publish.

## Install

Install via the vercel-labs/skills CLI:

```bash
# all four
npx skills add vercel-labs/agent-skills --skill 'agent-catalog-*'

# or individually
npx skills add vercel-labs/agent-skills --skill agent-catalog-author
```

The CLI detects your agent harness (Claude Code, Cursor, Codex, etc.) and installs to the right directory.
