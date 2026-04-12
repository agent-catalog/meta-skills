---
name: agent-catalog-validate
description: Use this skill to validate an agent-catalog source file (YAML or JSON) against the schema, check cross-references, and report human-readable errors. Wraps the reference server's verify command with model-friendly explanations.
---

# agent-catalog-validate

Use this skill to validate an agent-catalog manifest before deploying it. Catches schema violations, dangling cross-references, and structural issues early.

## When to use

- After authoring a draft catalog (manually, via `agent-catalog-author`, or via `agent-catalog-migrate`)
- Before deploying any catalog change to production
- When debugging why an existing catalog isn't being accepted by an SDK or harness

## How to use

### Step 1: Locate the source

Ask the publisher where their agent-catalog source file is. Common locations:
- `agent-catalog.yaml` at the repo root
- `infra/agent-catalog.yaml`
- `public/.well-known/agent-catalog.json` (the deployed canonical form)

### Step 2: Run the reference server's verify command

```bash
node node_modules/@agent-catalog/server/dist/cli.js verify <path-to-source>
```

If the publisher hasn't installed the reference server:

```bash
npx -y @agent-catalog/server verify <path-to-source>
```

### Step 3: Interpret the output

The verify command exits with:
- **Code 0** + `✓ Catalog is valid`: the catalog is compliant. Proceed to publish.
- **Code 1** + `✗ Catalog is invalid:` followed by error messages: there are issues to fix.

### Step 4: Translate the error messages for the publisher

The raw errors come from JSON Schema validation and the cross-reference checker. They look like:

- `(root): must have required property 'agentCatalogVersion'` → "You're missing the required `agentCatalogVersion: 1` field at the top of your YAML."
- `/origin: must match pattern "^https://"` → "Your `origin` field must use HTTPS, not HTTP."
- `/apis/0/format: must be equal to one of the allowed values` → "The `format` field on `apis[0]` must be one of: openapi, asyncapi, graphql, grpc, api-catalog."
- `/skills/0/requires/identity: dangling reference 'web-bot-auth-main'` → "The skill at `skills[0]` references identity scheme `web-bot-auth-main`, but you don't have an `auth.identity` entry with that ID. Either add the entry or remove the reference."

For each error, explain:
1. **What the error means** in plain language
2. **Which line/section of the YAML** is affected
3. **How to fix it** with a concrete suggestion

### Step 5: Re-run after fixing

After the publisher fixes the issues, re-run the verify command. Repeat until exit 0.

### Step 6: Hand off to publish

Once `✓ Catalog is valid`, tell the publisher to run `agent-catalog-publish` to deploy.

## Common issues

### Missing required fields

The schema requires `agentCatalogVersion` and `origin` at the top level. Every entry requires `id`, `name`, and `description`.

### `pinned: false` skill with a `hash` field

The schema enforces that unpinned skills cannot carry a hash. Remove the hash, or pin the skill to a commit SHA.

### `pinned: true` skill without a `commit` field

The schema enforces that pinned skills must declare which commit they pin to. Add the `commit` field.

### Cross-reference points to a missing auth entry

The `requires.identity` or `requires.authorization` field must reference an `id` that exists in `auth.identity[]` or `auth.authorization[]`. Either add the auth entry or remove the reference.

### Non-HTTPS URL in any entry

Every URL in the catalog must be HTTPS. Check `origin`, `apis[].url`, `mcps[].url`, `mcps[].card`, `agents[].card`, `docs[].url`, `auth.identity[].keyDirectoryUrl`, `auth.authorization[].metadataUrl`.

## Reference

- Spec: `spec/spec.md`
- Schema: `spec/schema/agent-catalog-v1.schema.json`
- Conformance vectors (positive and negative examples): `spec/conformance/`
