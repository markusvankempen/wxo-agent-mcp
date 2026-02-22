# Publishing an MCP Server

Procedure for creating, publishing to npm, and registering in the MCP Registry.

---

## Contents

1. [Create MCP Server](#1-create-mcp-server)
2. [Required Files for npm](#2-required-files-for-npm)
3. [Useful Files (Optional)](#3-useful-files-optional)
4. [Publish to npm](#4-publish-to-npm)
5. [Publish to MCP Registry](#5-publish-to-mcp-registry)
6. [Checklist](#6-checklist)
7. [Visibility](#7-visibility)

---

## 1. Create MCP Server

### Project structure

```
mcp-server-name/
├── src/
│   └── index.ts       # MCP server entry, tool handlers
├── dist/              # Compiled output (gitignore, built before publish)
├── package.json
├── tsconfig.json
├── .env.example
├── README.md
└── ...
```

### Minimum requirements

| Requirement | Notes |
|-------------|-------|
| **Shebang** | `#!/usr/bin/env node` at top of compiled entry (e.g. `dist/index.js`) |
| **MCP SDK** | Use `@modelcontextprotocol/sdk` (McpServer, StdioServerTransport) |
| **package.json** | `main`, `bin`, `mcpName` (for MCP Registry) |
| **Transport** | stdio (JSON-RPC over stdin/stdout) |

### package.json essentials

```json
{
  "name": "your-mcp-server",
  "version": "1.0.0",
  "mcpName": "io.github.your-username/your-server-name",
  "type": "module",
  "main": "dist/index.js",
  "bin": {
    "your-mcp-server": "dist/index.js"
  },
  "files": ["dist", "README.md", ".env.example"],
  "scripts": {
    "build": "tsc",
    "prepublishOnly": "npm run build"
  }
}
```

- **mcpName**: Required for MCP Registry. Format `io.github.username/repo-name` for GitHub auth.
- **bin**: Enables `npx your-mcp-server` without explicit path.
- **prepublishOnly**: Ensures `dist/` is built before publish.

---

## 2. Required Files for npm

| File | Purpose |
|------|---------|
| **package.json** | Name, version, dependencies, `files`, `bin` |
| **dist/** | Compiled JavaScript (in `files`) |
| **README.md** | Shown on npm package page |
| **LICENSE** | License file (e.g. Apache-2.0) |

### package.json `files` array

Only listed paths are included in the tarball. Omit = everything except `.gitignore` by default.

```json
"files": [
  "dist",
  "README.md",
  "DOCUMENTATION.md",
  "LICENSE",
  ".env.example",
  "examples",
  "resources"
]
```

---

## 3. Useful Files (Optional)

| File | Purpose |
|------|---------|
| **.env.example** | Documents required env vars |
| **examples/mcp.json** | Sample MCP client config (Cursor, VS Code) |
| **CONTRIBUTING.md** | Contribution guidelines |
| **DOCUMENTATION.md** | Detailed docs, screenshots |
| **resources/** | Icon (icon.png, icon.svg), screenshots |
| **server.json** | MCP Registry metadata (required for step 5; not shipped in npm) |

---

## 4. Publish to npm

### Prerequisites

- npm account: [npmjs.com](https://www.npmjs.com/)
- Logged in: `npm whoami` (or `npm login`)

### Steps

1. **Version**: `npm version patch` (or `minor` / `major`)
2. **Check**: `npm run check` (lint, format, build) if available
3. **Publish**:
   - **Unscoped**: `npm publish`
   - **Scoped** (e.g. `@username/pkg`): `npm publish --access public`

### Common issues

| Issue | Fix |
|-------|-----|
| 403 Forbidden (version exists) | Bump version with `npm version patch` |
| 403 Forbidden (scoped private) | Use `--access public` for public packages |
| Missing files | Add paths to `package.json` → `files` |
| Build errors | Fix `prepublishOnly`; ensure `tsc` runs before publish |

---

## 5. Publish to MCP Registry

The MCP Registry stores **metadata** only; the package must already be on npm.

### Prerequisites

- Package published on npm
- `mcpName` in package.json (e.g. `io.github.username/repo`)
- MCP Publisher CLI: `brew install mcp-publisher`

### Step 1: Create server.json

Run in your project directory:

```bash
mcp-publisher init
```

Or create manually. Minimal structure:

```json
{
  "$schema": "https://static.modelcontextprotocol.io/schemas/2025-12-11/server.schema.json",
  "name": "io.github.username/repo-name",
  "description": "Short description (max 100 chars)",
  "title": "Display Name",
  "websiteUrl": "https://github.com/username/repo#readme",
  "repository": {
    "url": "https://github.com/username/repo",
    "source": "github"
  },
  "version": "1.0.0",
  "packages": [
    {
      "registryType": "npm",
      "identifier": "your-package-name",
      "version": "1.0.0",
      "transport": { "type": "stdio" },
      "environmentVariables": [
        {
          "name": "API_KEY",
          "description": "Your API key",
          "isRequired": true,
          "format": "string",
          "isSecret": true
        }
      ]
    }
  ]
}
```

**Important:**

- `name` must match `mcpName` in package.json
- `version` and `packages[].version` must match package.json
- `description` max 100 characters

### Step 2: Log in

```bash
mcp-publisher login github
```

Follow the device flow in the browser. Token is stored locally.

### Step 3: Publish

```bash
mcp-publisher publish
```

### After npm version bumps

Whenever you publish a new npm version:

1. Update `version` in `server.json` (root and `packages[].version`)
2. Run `mcp-publisher publish`

---

## 6. Checklist

### Creating an MCP server

- [ ] Project with `src/`, `package.json`, `tsconfig.json`
- [ ] MCP SDK (`@modelcontextprotocol/sdk`)
- [ ] Shebang in entry file: `#!/usr/bin/env node`
- [ ] `package.json`: `main`, `bin`, `mcpName`, `files`, `prepublishOnly`
- [ ] `.env.example` with required env vars

### Publishing to npm

- [ ] `npm run build` succeeds
- [ ] `package.json` `files` includes: dist, README, LICENSE
- [ ] `npm version patch` (or minor/major)
- [ ] `npm publish` (or `npm publish --access public` for scoped)

### Publishing to MCP Registry

- [ ] Package already on npm
- [ ] `server.json` created (`mcp-publisher init` or manual)
- [ ] `server.json` `name` = package `mcpName`
- [ ] `server.json` `version` = package version
- [ ] `server.json` `description` ≤ 100 chars
- [ ] `mcp-publisher login github` (if not logged in)
- [ ] `mcp-publisher publish`

---

## 7. Visibility

### npm

| Action | Purpose |
|--------|---------|
| **Badges** | Add to README: version, downloads, license (shields.io) |
| **homepage** | `package.json` → links to docs on npm page |
| **bugs** | `package.json` → issue tracker link |
| **keywords** | Good keywords improve npm search (mcp, watsonx, cursor, etc.) |
| **description** | Clear, concise – shown in search results |

### MCP Registry

- Publish with `mcp-publisher publish` so the server appears at [registry.modelcontextprotocol.io](https://registry.modelcontextprotocol.io)
- Used by Cursor, Claude Desktop, and other clients that browse the registry

### Directories and lists (submit manually)

| Platform | URL | Notes |
|----------|-----|-------|
| **MCP Awesome** | [mcp-awesome.com](https://mcp-awesome.com) | 1200+ servers, by category |
| **MCP Servers List** | [mcpserverslist.com](https://www.mcpserverslist.com) | 500+ servers |
| **Antigravity Codes** | [antigravity.codes/mcp](https://antigravity.codes/mcp) | Paid sponsorship; [advertise](https://antigravity.codes/advertise) ($29–99/mo). Directory may auto-index from MCP Registry |
| **Cursor Directory** | [cursor.directory/mcp](https://cursor.directory/mcp) | 72k+ members; [post a MCP](https://cursor.directory/mcp/new) |
| **Awesome MCP (Glama)** | [glama.ai/mcp/servers](https://glama.ai/mcp/servers) | From punkpeye/awesome-mcp-servers |
| **MCP Servers Org** | [mcpservers.org](https://mcpservers.org) | From wong2/awesome-mcp-servers |

Check each site for "Add server" or "Submit" and follow their process.

### Cursor Directory

[cursor.directory](https://cursor.directory) is a community hub for Cursor (72k+ members) with rules, MCPs, jobs, and a board. **To post your MCP:**

1. Go to [cursor.directory/mcp/new](https://cursor.directory/mcp/new)
2. Fill in the form (name, description, setup/config, etc.)
3. Submit – your MCP will be listed and reach 250,000+ monthly active developers

Individual MCP pages use slugs like `cursor.directory/mcp/mailtrap` or `cursor.directory/mcp/wxo-agent-mcp` (based on your submission).

### Antigravity Codes – paid sponsorship

[antigravity.codes/advertise](https://antigravity.codes/advertise) is a **paid** platform. There is no free "add your server" form.

| Tier  | Price  | Includes                                             |
|-------|--------|------------------------------------------------------|
| Indie | $29/mo | Rotating placement, 2x inline ads                    |
| Pro   | $49/mo | Featured spot, section banner, 4x inline ads          |
| Growth| $99/mo | 6x inline ads, **featured MCP listing**, navbar banner |

**How to advertise:**
1. Go to [antigravity.codes/advertise](https://antigravity.codes/advertise)
2. Choose a tier (Get Started / Choose Pro / Go Growth)
3. Or use "Contact Sales" / "Contact Us Today" for custom options

The MCP directory at [antigravity.codes/mcp](https://antigravity.codes/mcp) may auto-include servers from the MCP Registry; publishing to the registry can provide free exposure there.

### GitHub

- **Topics:** Add `mcp`, `model-context-protocol`, `watsonx`, `watson-orchestrate`, `cursor`, `vscode`, `langflow` to repo topics
- **README:** Clear title, badges, quick start, screenshots
- **Releases:** Tag versions (e.g. `v1.0.5`) for changelog and npm alignment

---

### Links

- **MCP Registry**: [registry.modelcontextprotocol.io](https://registry.modelcontextprotocol.io)
- **Quickstart**: [modelcontextprotocol.io/registry/quickstart](https://modelcontextprotocol.io/registry/quickstart)
- **Schema**: `https://static.modelcontextprotocol.io/schemas/2025-12-11/server.schema.json`
