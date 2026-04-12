---
name: agent-catalog-migrate
description: Use this skill when a publisher already publishes some standard well-known files (api-catalog, mcp.json, agent-card.json, oauth-authorization-server, llms.txt, etc.) and wants to aggregate them into an agent-catalog manifest without rewriting from scratch.
---

# agent-catalog-migrate

Use this skill when a publisher already exposes standard well-known files at their origin and wants to aggregate them into a single agent-catalog manifest. This is the most directly useful skill for sites with existing infrastructure — it discovers what they already have and assembles a draft catalog that points at it, rather than asking them to re-author everything.

## When to use

- The publisher already has one or more of: `/.well-known/api-catalog`, `/.well-known/mcp.json`, `/.well-known/agent-card.json`, `/.well-known/oauth-authorization-server`, `/.well-known/openid-configuration`, `/.well-known/web-bot-auth`, `/llms.txt`, `/AGENTS.md`, `/security.txt`, `/sitemap.xml`
- The publisher wants the umbrella catalog to point at these files rather than duplicate their content
- The publisher wants to know what's already discoverable on their origin

## When NOT to use

- The publisher has nothing existing — use **`agent-catalog-author`** for the zero-to-draft flow.

## How to use

### Step 1: Gather the origin

Ask the publisher for their HTTPS origin (e.g., `https://acme.example`).

### Step 2: Probe each well-known location

For each of the standard well-known paths below, fetch it from the publisher's origin and check if it exists. Use the bash tool:

```bash
ORIGIN=https://acme.example
for path in \
  ".well-known/api-catalog" \
  ".well-known/mcp.json" \
  ".well-known/agent-card.json" \
  ".well-known/oauth-authorization-server" \
  ".well-known/openid-configuration" \
  ".well-known/web-bot-auth" \
  "llms.txt" \
  "AGENTS.md" \
  "security.txt" \
  "sitemap.xml" \
  ; do
  status=$(curl -s -o /dev/null -w "%{http_code}" "$ORIGIN/$path")
  echo "$status $path"
done
```

Record which return 2xx (the publisher has them) and which return 404 (they don't).

### Step 3: For multi-A2A-agent origins, also probe agent paths

If the publisher told you about specific A2A agents at non-root paths, probe those too:

```bash
curl -sf https://acme.example/agents/support/.well-known/agent-card.json > /tmp/support-card.json
curl -sf https://acme.example/agents/billing/.well-known/agent-card.json > /tmp/billing-card.json
```

### Step 4: Build the draft catalog

For each existing well-known file, add an entry of the corresponding type. Map:

| Well-known file | Catalog entry type | Notes |
|---|---|---|
| `/.well-known/api-catalog` | `apis[]` with `format: api-catalog` | One entry pointing at the api-catalog URL |
| `/.well-known/mcp.json` | `mcps[]` with `card` field | Read the card to extract transport, name, description |
| `/.well-known/agent-card.json` | `agents[]` | Read the card to extract name and description; the `card` field is the well-known URL |
| `/.well-known/oauth-authorization-server` | `auth.authorization[]` with `scheme: oauth2` and `metadataUrl` | One entry, conventionally `id: oauth-main` |
| `/.well-known/openid-configuration` | `auth.authorization[]` with `scheme: oauth2` and `metadataUrl` | OIDC is a superset of OAuth 2.0; treat as OAuth |
| `/.well-known/web-bot-auth` | `auth.identity[]` with `scheme: web-bot-auth` and `keyDirectoryUrl` | One entry, conventionally `id: web-bot-auth-main` |
| `/llms.txt` | `docs[]` with `purpose: index` | Estimate token count from `wc -w` |
| `/AGENTS.md` | `docs[]` with `purpose: agents-md` | Same |

### Step 5: Read each well-known file's content to populate descriptions

For each entry, fetch the well-known file and extract a description from its content:
- API catalogs: read the linkset and produce a one-sentence summary of what APIs are listed
- MCP cards: use the card's `description` field
- Agent cards: use the card's `description` field
- OAuth metadata: standard description ("user-scoped OAuth 2.0 access via authorization-code flow with PKCE")
- llms.txt: read the first H1 + first paragraph

Apply the model-native register style (see `agent-catalog-author` skill).

### Step 6: Cross-reference auth

If you found both an identity entry AND an OAuth entry, add `requires: { identity: web-bot-auth-main, authorization: oauth-main }` to every API and MCP entry that you can confirm requires authentication. If unsure, leave `requires` off — the publisher can fill in.

### Step 7: Tell the publisher what's missing

After assembling the draft, list things the publisher does NOT have but COULD add:
- "You don't publish a `/.well-known/api-catalog` — consider creating one to consolidate your APIs"
- "You don't publish any SKILL.md skills — consider creating some via the vercel-labs/skills CLI"
- "You don't have a Web Bot Auth key directory — consider setting one up to accept cryptographically-identified agents"

### Step 8: Hand off

Tell the publisher to run `agent-catalog-validate` to check the draft, then `agent-catalog-publish` to deploy.

## Example output

```yaml
agentCatalogVersion: 1
origin: https://acme.example
name: Acme Corp
description: Auto-discovered from existing well-known files at acme.example.

apis:
  - id: all-apis
    name: Acme APIs
    description: All Acme APIs, indexed via the existing api-catalog at this origin.
    format: api-catalog
    url: https://acme.example/.well-known/api-catalog

mcps:
  - id: discovered-mcp
    name: (from /.well-known/mcp.json)
    description: (from the card's description field)
    transport: http
    url: (from the card's endpoint field)
    card: https://acme.example/.well-known/mcp.json
    requires:
      authorization: oauth-main

auth:
  identity:
    - id: web-bot-auth-main
      scheme: web-bot-auth
      description: Web Bot Auth key directory discovered at /.well-known/web-bot-auth.
      keyDirectoryUrl: https://acme.example/.well-known/web-bot-auth
  authorization:
    - id: oauth-main
      scheme: oauth2
      description: OAuth 2.0 metadata discovered at /.well-known/oauth-authorization-server.
      metadataUrl: https://acme.example/.well-known/oauth-authorization-server
```

## Reference

- Same as `agent-catalog-author`.
