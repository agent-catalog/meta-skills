# agent-catalog meta-skills

Four SKILL.md skills that help publishers create, migrate, validate, and deploy agent-catalog manifests for their own sites.

## The four skills

| Skill | When to use |
|---|---|
| **`agent-catalog-author`** | Zero to draft. Walks through every entry type, asks the publisher what they have, produces a YAML source file. |
| **`agent-catalog-migrate`** | The publisher already has standard well-known files. Sniffs the origin for them and produces a draft that aggregates. |
| **`agent-catalog-validate`** | Validates a draft against the schema, checks cross-references, reports model-friendly errors. |
| **`agent-catalog-publish`** | Helps the publisher choose a deployment target and produces deploy-ready artifacts. |

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

Each skill is independent — a publisher who already has a draft can skip straight to validate, and one who already has a validated catalog can skip to publish.

## Installation

These skills are distributed via the `vercel-labs/agent-skills` collection (placeholder until actual mirror-publication). Install via the vercel-labs/skills CLI:

```bash
# All four
npx skills add vercel-labs/agent-skills --skill 'agent-catalog-*'

# Or one at a time
npx skills add vercel-labs/agent-skills --skill agent-catalog-author
npx skills add vercel-labs/agent-skills --skill agent-catalog-migrate
npx skills add vercel-labs/agent-skills --skill agent-catalog-validate
npx skills add vercel-labs/agent-skills --skill agent-catalog-publish
```

The vercel-labs/skills CLI auto-detects 45+ supported agent harnesses (Claude Code, Cursor, Codex, OpenCode, Cline, Goose, Continue, Windsurf, etc.) and installs the skills into the right per-harness directory.

## Dogfood

These four skills are themselves SKILL.md files in vercel-labs/skills format, distributed via the same ecosystem the agent-catalog spec recommends. The project's deliverable IS the spec it advocates for.
