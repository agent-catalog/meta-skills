---
name: agent-catalog-author
description: Use this skill when a publisher wants to create an agent-catalog manifest from scratch for their site, walking through every entry type with model-native register prose, producing a compliant YAML source file.
---

# agent-catalog-author

Use this skill when a publisher wants to create an agent-catalog v1 manifest from scratch and doesn't yet have one. This skill walks through every entry type, asks the publisher what they have, and produces a YAML source file that the reference server can build and serve.

## When to use

- The publisher is starting from zero (no existing well-known files for agents)
- The publisher knows their site has APIs, MCPs, agents, skills, SDKs, or docs but doesn't know how to expose them as an agent-catalog
- The publisher wants the model-native description style enforced for them rather than writing it manually

## When NOT to use

- The publisher already has existing well-known files (`api-catalog`, `mcp.json`, `agent-card.json`, `oauth-authorization-server`, `llms.txt`, etc.). Use **`agent-catalog-migrate`** instead — it sniffs the origin for those files and produces a draft that aggregates them.
- The publisher has a draft and wants to validate it. Use **`agent-catalog-validate`**.
- The publisher has a validated draft and wants to deploy it. Use **`agent-catalog-publish`**.

## How to use

### Step 1: Gather publisher metadata

Ask the publisher:
- What's the HTTPS origin? (e.g., `https://acme.example`)
- Organization name?
- A one-sentence description of what the site does for agents?
- A contact email for `agents@<domain>`?

### Step 2: Walk the seven entry types

For each entry type, ask the publisher whether they have any. If yes, gather the details. Use the prompts below.

#### `apis[]` — REST/GraphQL/gRPC APIs

Ask:
- Do you have any HTTP APIs that agents should call?
- For each: name, one-sentence description, format (openapi / asyncapi / graphql / grpc), URL to the spec, auth requirements.
- If they publish many APIs, ask if they have an RFC 9727 `/.well-known/api-catalog` — if yes, use a single entry with `format: api-catalog`.

#### `mcps[]` — MCP servers

Ask:
- Do you expose any MCP servers? (Model Context Protocol)
- For each: name, description, transport (stdio / http / sse), URL or command, install instructions.
- If they have an SEP-1649 server card, capture the `card` URL.

#### `agents[]` — A2A agents

Ask:
- Do you expose any A2A (Agent2Agent) agents? Each one publishes its own `/.well-known/agent-card.json`.
- For each: name, description, the URL to the agent-card.json.

#### `skills[]` — installable SKILL.md skills

Ask:
- Do you publish any SKILL.md skills (vercel-labs/skills format)?
- For each: name, description, vercel-labs source string, pinned commit SHA (strongly recommended).

#### `sdks[]` — client libraries

Ask:
- Do you publish official client libraries?
- For each: language, package coordinate (`npm:`, `pypi:`, `cargo:`, etc.), docs URL.

#### `docs[]` — markdown for context ingestion

Ask:
- Do you have any markdown documentation that agents should ingest into context (rate-limit policies, getting-started guides, error code references)?
- For each: name, description, URL, approximate token count, purpose tag.

#### `auth.identity[]` and `auth.authorization[]`

Ask:
- Do you require agents to identify themselves cryptographically? If yes, capture as a Web Bot Auth identity entry with `keyDirectoryUrl`.
- Do you accept anonymous calls for some endpoints? If yes, add an `anonymous` identity entry.
- Do you use OAuth 2.0? If yes, point at the RFC 8414 `/.well-known/oauth-authorization-server` URL.
- Do you accept API keys? If yes, capture the header name and where users get keys.

### Step 3: Apply the model-native register style

For every `description` and `whenToUse` field, rewrite the prose into:
- **Second person, imperative**: "Use this when..." not "This is for..."
- **Specific, not abstract**: "Query and modify customer billing state" not "A flexible billing API"
- **No marketing words**: avoid "powerful", "comprehensive", "state-of-the-art", "best-in-class"

Bad: ✗ "Our state-of-the-art billing API empowers your workflow."
Good: ✓ "Query and modify customer billing state, invoices, and payment methods."

### Step 4: Write the YAML

Produce the YAML source file at the publisher's chosen path (typically `agent-catalog.yaml` in their repo root or `infra/agent-catalog.yaml`). Use the gold-standard example at `examples/bundle/agent-catalog.yaml` in the agent-catalog repo as your reference for structure.

Required top-level fields:
- `agentCatalogVersion: 1`
- `origin: https://...`

Strongly recommended:
- `name`, `description`, `publisher`

Then the entry collections you populated.

### Step 5: Hand off to the next skill

Tell the publisher:
- "Run `agent-catalog-validate` next to check your draft"
- Once validated: "Run `agent-catalog-publish` to deploy"

## Example

After running this skill, the publisher should have a YAML file like:

```yaml
agentCatalogVersion: 1
origin: https://acme.example
name: Acme Corp
description: Acme exposes billing and support APIs and a self-service knowledge base.
publisher:
  name: Acme Inc.
  contact: agents@acme.example
apis:
  - id: billing-api
    name: Billing API
    description: Query and modify customer billing state, invoices, and payment methods.
    format: openapi
    url: https://acme.example/openapi.yaml
    requires:
      identity: web-bot-auth-main
      authorization: oauth-main
auth:
  identity:
    - id: web-bot-auth-main
      scheme: web-bot-auth
      description: Acme accepts agent identity via Web Bot Auth.
      keyDirectoryUrl: https://acme.example/.well-known/web-bot-auth
  authorization:
    - id: oauth-main
      scheme: oauth2
      description: User-scoped access via OAuth 2.0 authorization-code flow.
      metadataUrl: https://acme.example/.well-known/oauth-authorization-server
```

## Reference

- Spec: `https://github.com/<owner>/agent-catalog/blob/main/spec/spec.md`
- Schema: `https://github.com/<owner>/agent-catalog/blob/main/spec/schema/agent-catalog-v1.schema.json`
- Best Practices (style guide): `https://github.com/<owner>/agent-catalog/blob/main/spec/best-practices.md`
- Gold-standard example: `https://github.com/<owner>/agent-catalog/blob/main/examples/bundle/agent-catalog.yaml`
