# 🔄 Reusable GitHub Actions for OpenAPI Workflows

**Public, generic, and easy-to-use GitHub Actions for common OpenAPI workflows.**

Perfect for teams managing OpenAPI specs across multiple repositories who want to automate syncing, bundling, and SDK publishing.

---

## 🎯 What's Included

### 1. OpenAPI Sync
Download and validate OpenAPI specs from a remote source.

### 2. OpenAPI Bundle
Combine multiple OpenAPI YAML files into a single bundled spec using Redocly CLI.

### 3. npm Publish
Regenerate SDK code, bump version, build, and publish to npm automatically.

---

## 🚀 Quick Start

### Example: Auto-sync OpenAPI spec from backend

In your consuming repo (e.g., SDK or docs site):

```yaml
# .github/workflows/sync-openapi.yml
name: Sync OpenAPI Spec

on:
  repository_dispatch:
    types: [openapi-updated]
  workflow_dispatch:

jobs:
  sync:
    uses: beel-es/github-actions/.github/workflows/sync-from-openapi.yml@main
    with:
      spec-url: 'https://raw.githubusercontent.com/your-org/backend/main/openapi.yaml'
      output-file: 'openapi.yaml'
```

**That's it!** 🎉

---

## 📦 Actions Reference

### `openapi-sync`

Download and optionally validate an OpenAPI spec.

**Inputs:**
- `spec-url` (required) - URL to download the spec from
- `spec-hash` (optional) - Expected SHA256 hash for validation
- `output-file` (optional, default: `openapi.yaml`) - Where to save the spec
- `verify-hash` (optional, default: `true`) - Whether to verify hash

**Outputs:**
- `spec-path` - Path to the downloaded spec
- `spec-hash` - Actual SHA256 hash
- `has-changes` - Whether the spec changed vs current version

**Example:**

```yaml
- uses: beel-es/github-actions/openapi-sync@main
  with:
    spec-url: 'https://api.example.com/openapi.yaml'
    output-file: 'api-spec.yaml'
```

---

### `openapi-bundle`

Bundle multiple OpenAPI YAML files into one using Redocly CLI.

**Inputs:**
- `input-file` (required) - Main OpenAPI file (references other files with `$ref`)
- `output-file` (optional, default: `bundled.yaml`) - Output bundled spec
- `remove-unused-components` (optional, default: `false`) - Clean up unused schemas
- `dereference` (optional, default: `false`) - Inline all `$ref` (not recommended for large specs)

**Outputs:**
- `bundled-path` - Path to the bundled spec
- `bundle-hash` - SHA256 hash of bundled file

**Example:**

```yaml
- uses: beel-es/github-actions/openapi-bundle@main
  with:
    input-file: 'src/openapi/main.yaml'
    output-file: 'dist/openapi-bundled.yaml'
    remove-unused-components: true
```

---

### `npm-publish`

**Composite action** (use inside a job, not as a standalone workflow)

Regenerate code, build, bump version, and publish to npm.

**Inputs:**
- `generate-script` (optional) - npm script to run for code generation
- `build-script` (optional, default: `build`) - npm script to build
- `skip-tests` (optional, default: `false`) - Skip running tests
- `npm-token` (required) - npm authentication token

**Outputs:**
- `published` - Whether package was published
- `version` - New version number

**Example:**

```yaml
- uses: beel-es/github-actions/npm-publish@main
  with:
    generate-script: 'generate'
    build-script: 'build'
    npm-token: ${{ secrets.NPM_TOKEN }}
```

---

## 🌐 Reusable Workflows

### `sync-from-openapi.yml`

Complete workflow to sync OpenAPI spec from a remote URL.

**Inputs:**
- `spec-url` - URL to download spec
- `spec-hash` - Expected hash (optional)
- `output-file` - Output filename (default: `openapi.yaml`)
- `commit-message` - Custom commit message (optional)

**Outputs:**
- `has-changes` - Whether changes were detected
- `spec-hash` - Hash of downloaded spec

**Usage:**

```yaml
jobs:
  sync:
    uses: beel-es/github-actions/.github/workflows/sync-from-openapi.yml@main
    with:
      spec-url: 'https://example.com/openapi.yaml'
      output-file: 'my-spec.yaml'
```

---

### `bundle-openapi.yml`

Complete workflow to bundle OpenAPI files and commit the result.

**Inputs:**
- `input-file` - Main OpenAPI file
- `output-file` - Bundled output (default: `bundled.yaml`)
- `remove-unused-components` - Clean unused schemas (default: `false`)
- `commit-message` - Custom commit message (optional)

**Outputs:**
- `has-changes` - Whether bundled spec changed
- `bundle-hash` - SHA256 hash of bundle

**Usage:**

```yaml
jobs:
  bundle:
    uses: beel-es/github-actions/.github/workflows/bundle-openapi.yml@main
    with:
      input-file: 'src/openapi/main.yaml'
      output-file: 'dist/bundled.yaml'
      remove-unused-components: true
```

---

### `publish-npm-sdk.yml`

Complete workflow to regenerate SDK, bump version, and publish to npm.

**Inputs:**
- `node-version` - Node.js version (default: `20`)
- `generate-script` - npm script for generation (optional)
- `build-script` - npm build script (default: `build`)
- `skip-tests` - Skip tests (default: `false`)

**Secrets:**
- `NPM_TOKEN` (required) - npm authentication token

**Outputs:**
- `published` - Whether package was published
- `version` - New version number

**Usage:**

```yaml
jobs:
  publish:
    uses: beel-es/github-actions/.github/workflows/publish-npm-sdk.yml@main
    with:
      generate-script: 'generate'
      build-script: 'build'
    secrets:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

---

## 🎨 Real-World Examples

### Example 1: Backend triggers sync to SDK

**Backend repo** (`backend/.github/workflows/sync-openapi.yml`):

```yaml
name: Sync OpenAPI to dependent repos

on:
  push:
    branches: [main]
    paths:
      - 'openapi/**'

jobs:
  trigger:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Get spec hash
        id: hash
        run: echo "hash=$(sha256sum openapi/bundled.yaml | cut -d' ' -f1)" >> $GITHUB_OUTPUT
      
      - name: Trigger SDK repo
        uses: peter-evans/repository-dispatch@v2
        with:
          token: ${{ secrets.SYNC_TOKEN }}
          repository: my-org/node-sdk
          event-type: openapi-updated
          client-payload: |
            {
              "spec_url": "https://raw.githubusercontent.com/${{ github.repository }}/main/openapi/bundled.yaml",
              "spec_hash": "${{ steps.hash.outputs.hash }}"
            }
```

**SDK repo** (`node-sdk/.github/workflows/auto-regenerate.yml`):

```yaml
name: Auto-regenerate SDK

on:
  repository_dispatch:
    types: [openapi-updated]

jobs:
  sync:
    uses: beel-es/github-actions/.github/workflows/sync-from-openapi.yml@main
    with:
      spec-url: ${{ github.event.client_payload.spec_url }}
      spec-hash: ${{ github.event.client_payload.spec_hash }}
  
  publish:
    needs: sync
    if: needs.sync.outputs.has-changes == 'true'
    uses: beel-es/github-actions/.github/workflows/publish-npm-sdk.yml@main
    with:
      generate-script: 'generate'
    secrets:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

### Example 2: Bundle OpenAPI on every commit

**Backend repo** (`backend/.github/workflows/bundle.yml`):

```yaml
name: Bundle OpenAPI

on:
  push:
    branches: [main, develop]
    paths:
      - 'src/openapi/**'

jobs:
  bundle:
    uses: beel-es/github-actions/.github/workflows/bundle-openapi.yml@main
    with:
      input-file: 'src/openapi/main.yaml'
      output-file: 'openapi-bundled/api.yaml'
      remove-unused-components: true
      commit-message: 'chore: auto-bundle OpenAPI spec'
```

---

## 🔐 Security

These actions **do not** store or expose secrets. They are designed to be used with GitHub's secret management:

- **Secrets are passed by the caller** (your workflow)
- Actions only use secrets in the scope of a single job
- No secrets are logged or persisted

---

## 📝 Versioning

Use version tags in your workflows:

- `@main` - Latest version (may have breaking changes)
- `@v1` - Major version 1 (backwards compatible)
- `@v1.2.3` - Specific version (pinned)

**Recommended:**

```yaml
uses: beel-es/github-actions/.github/workflows/sync-from-openapi.yml@v1
```

---

## 🤝 Contributing

This is a **public, community-friendly** project. PRs welcome!

**Ideas for contributions:**
- Add support for other bundling tools (swagger-cli, openapi-merge)
- Add validation step with Spectral
- Add diff comparison between versions
- Support for multiple output formats (JSON, YAML)

---

## 📄 License

MIT

---

## 🐝 Created by BeeL Team

Built for our own OpenAPI workflows, shared with the community.

**Need help?** Open an issue on GitHub.
