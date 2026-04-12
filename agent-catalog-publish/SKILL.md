---
name: agent-catalog-publish
description: Use this skill to deploy a validated agent-catalog source file to a publisher's infrastructure. Walks through choosing a deployment target (static site, S3/R2 bucket, well-known route on an existing server) and produces deploy-ready artifacts.
---

# agent-catalog-publish

Use this skill once an agent-catalog has been authored and validated, to actually deploy it so agents can fetch it.

## When to use

- After `agent-catalog-validate` has reported the catalog as compliant
- The publisher needs to make the catalog reachable at `/.well-known/agent-catalog.json` on their origin

## How to use

### Step 1: Choose a deployment target

Ask the publisher how their site is hosted. Common options:

- **Static site / CDN** (Netlify, Vercel, Cloudflare Pages, GitHub Pages): drop the canonical JSON at `/.well-known/agent-catalog.json` in the static output. No server needed.
- **S3 / R2 / GCS bucket**: upload `agent-catalog.json` and configure the bucket to serve it at the `.well-known` path with `Content-Type: application/json`.
- **Existing Node/Python/Go server**: add a route serving the catalog file. For Node, mount a static handler on `/.well-known/agent-catalog.json`.
- **Reference server**: run `agent-catalog serve` as a sidecar to your existing infrastructure. This is the simplest but adds a process to manage.

### Step 2: Build the deploy artifacts

Use the reference server's `build` command:

```bash
node node_modules/@agent-catalog/server/dist/cli.js build \
  --source path/to/agent-catalog.yaml \
  --out-dir path/to/output
```

(Or via `npx -y @agent-catalog/server build ...` if not installed.)

This produces:
- `output/agent-catalog.json` — the canonical JSON wire format
- `output/agent-catalog.md` — the markdown projection (optional, served via content negotiation)

### Step 3: Deploy per target

#### Static site / CDN

Drop `agent-catalog.json` into the static site's source under `public/.well-known/` (the exact path depends on the framework — Next.js uses `public/`, Astro uses `public/`, Vite uses `public/`, etc.). The CDN will serve it at `https://<origin>/.well-known/agent-catalog.json`.

The markdown projection is optional. If you want to serve it via content negotiation, you'll need a server that can route based on `Accept` header — pure static CDNs typically can't do this. Skip the markdown and serve only JSON.

#### S3 / R2 bucket

```bash
aws s3 cp output/agent-catalog.json s3://your-bucket/.well-known/agent-catalog.json \
  --content-type "application/json" \
  --cache-control "max-age=3600"
```

For R2:
```bash
wrangler r2 object put your-bucket/.well-known/agent-catalog.json \
  --file output/agent-catalog.json \
  --content-type application/json
```

Configure the bucket's static-website settings to expose the `.well-known` path.

#### Existing Node server

Mount a static handler on the well-known path:

```typescript
import { readFile } from "node:fs/promises";

app.get("/.well-known/agent-catalog.json", async (c) => {
  const json = await readFile("./agent-catalog.json", "utf-8");
  return c.text(json, 200, {
    "Content-Type": "application/json; charset=utf-8",
    "Cache-Control": "max-age=3600"
  });
});
```

For markdown content negotiation:
```typescript
app.get("/.well-known/agent-catalog.json", async (c) => {
  const accept = c.req.header("Accept") ?? "application/json";
  if (accept.includes("text/markdown")) {
    const md = await readFile("./agent-catalog.md", "utf-8");
    return c.text(md, 200, { "Content-Type": "text/markdown; charset=utf-8" });
  }
  const json = await readFile("./agent-catalog.json", "utf-8");
  return c.text(json, 200, { "Content-Type": "application/json; charset=utf-8" });
});
```

#### Reference server as sidecar

```bash
node node_modules/@agent-catalog/server/dist/cli.js serve \
  --source path/to/agent-catalog.yaml \
  --port 8080
```

Then put a reverse proxy (nginx, Cloudflare Worker, Caddy) in front of it that routes `/.well-known/agent-catalog.json` to the sidecar's port.

### Step 4: Set the Link header (optional but recommended)

If you have control over the main site's HTTP responses, add a `Link` header to each response advertising the catalog:

```
Link: </agent-catalog.json>; rel="agent-catalog"
```

The header takes precedence over the well-known fallback, so agents that hit any URL on your origin discover the catalog without a separate probe round-trip. This is the SHOULD-tier recommendation in the spec.

### Step 5: Verify the live deployment

After deploying:

```bash
node node_modules/@agent-catalog/server/dist/cli.js verify https://your-origin.example
```

Expected: `✓ Catalog is valid`. The verify command fetches from the live URL using the same discovery handshake an agent would, so this is the most realistic check.

### Step 6: Set up CI to re-deploy on source changes

If the source file lives in git, add a CI job that runs `agent-catalog build` + uploads on every push to main. Example GitHub Actions snippet:

```yaml
name: deploy-agent-catalog
on:
  push:
    branches: [main]
    paths: [agent-catalog.yaml]
jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npx -y @agent-catalog/server build --source agent-catalog.yaml --out-dir dist
      - run: aws s3 cp dist/agent-catalog.json s3://your-bucket/.well-known/agent-catalog.json --content-type application/json
```

## Verification checklist

After deployment, verify all of these:

- [ ] `curl -sI https://your-origin.example/.well-known/agent-catalog.json` returns 200 + `Content-Type: application/json`
- [ ] `curl -s https://your-origin.example/.well-known/agent-catalog.json | jq .agentCatalogVersion` returns `1`
- [ ] `agent-catalog verify https://your-origin.example` exits 0
- [ ] If markdown projection is served, `curl -H "Accept: text/markdown" ...` returns the markdown
- [ ] If the Link header is set, `curl -sI https://your-origin.example/` includes `Link: <...>; rel="agent-catalog"`

## Reference

- Reference server: `server/`
- Spec § Discovery: `spec/spec.md`
- Spec § Trust and integrity: `spec/spec.md`
