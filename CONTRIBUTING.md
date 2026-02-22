# Contributing to wxo-agent-mcp

> **Note:** Contributions are welcome!

wxo-agent-mcp is an open-source MCP (Model Context Protocol) server that invokes a single Watson Orchestrate agent. This page describes how you can contribute.

**Author:** Markus van Kempen (markus.van.kempen@gmail.com)  
**License:** Apache-2.0

## Before You Start

- Read the [README](./README.md) and [DOCUMENTATION.md](./DOCUMENTATION.md) to understand the MCP server and its tools.

## MCP Tools

The server exposes two tools:

| Tool             | Description                                                                           |
| ---------------- | ------------------------------------------------------------------------------------- |
| **invoke_agent** | Send a message to the configured Watson Orchestrate agent. Returns the agent's reply. |
| **get_agent**    | Get agent details (name, description, tools, instructions).                           |

When contributing, test your changes by exercising both tools—see [Testing with the Tools](#testing-with-the-tools) below.

## Style and Lint

This project uses the following tools:

- [ESLint](https://eslint.org/) — Linting for TypeScript
- [Prettier](https://prettier.io/) — Code formatter
- [TypeScript](https://www.typescriptlang.org/) — Type checking via `tsc`

### Available Scripts

| Command                | Description                                                     |
| ---------------------- | --------------------------------------------------------------- |
| `npm run build`        | Compile TypeScript to `dist/`                                   |
| `npm run start`        | Run the MCP server (stdio)                                      |
| `npm run test:verify`  | Run verification: `get_agent` + `invoke_agent` against your env |
| `npm run lint`         | Run ESLint on `src/`                                            |
| `npm run lint:fix`     | ESLint with auto-fix                                            |
| `npm run format`       | Format all files with Prettier                                  |
| `npm run format:check` | Check formatting without modifying                              |
| `npm run check`        | Lint + format check + build (full CI check)                     |

## Set Up a Development Environment

1. Clone the repo:

    ```sh
    git clone https://github.com/markusvankempen/wxo-agent-mcp.git
    cd wxo-agent-mcp
    ```

    Or from the monorepo:

    ```sh
    cd watsonx-orchestrate-devkit/packages/wxo-agent-mcp
    ```

2. Install dependencies:

    ```sh
    npm install
    ```

3. Copy and configure env:

    ```sh
    cp .env.example .env
    # Edit .env with WO_API_KEY, WO_INSTANCE_URL, WO_AGENT_ID
    ```

4. Run the full check:

    ```sh
    npm run check
    ```

5. Verify connectivity:
    ```sh
    npm run test:verify
    ```

## Testing with the Tools

Use the MCP tools to validate your changes end-to-end.

### 1. Terminal (test:verify)

```sh
npm run test:verify
# Or with a custom question:
npm run test:verify -- -ask "What is the weather in Amsterdam?"
```

### 2. Cursor / VS Code / Antigravity

1. Configure the MCP server (see `examples/mcp.json` or `examples/antigravity-mcp.json`).
2. Open Copilot Chat (VS Code: Ctrl+Shift+I) or Cursor chat.
3. Ask: _Use invoke_agent to ask: What can you do?_
4. Or: _Use get_agent to show the agent details_

### 3. Langflow

Add an MCP Tools component, use STDIO, set Command=`npx`, Args=`["-y", "wxo-agent-mcp"]`, and env vars. See [DOCUMENTATION.md#testing-in-langflow](./DOCUMENTATION.md#testing-in-langflow).

## Conventional Commits

We use [Conventional Commits](https://www.conventionalcommits.org/) for commit messages:

```
<type>(<scope>): <subject>
```

**Types:** `feat`, `fix`, `chore`, `docs`, `style`, `refactor`, `perf`, `test`

**Scopes:** `mcp`, `auth`, `config`, `tools`, `deps`, `docs`

Examples:

```
feat(tools): add optional conversation_id to invoke_agent
fix(auth): handle IAM token refresh
docs(readme): add Langflow setup
chore(deps): upgrade @modelcontextprotocol/sdk
```

## Issues and Pull Requests

- Open an issue before significant work to discuss the approach.
- Use Draft PRs with `[WIP]` for work-in-progress.
- All PRs must pass `npm run check` before merging.

## Project Structure

```
wxo-agent-mcp/
├── src/
│   ├── index.ts   # MCP server, invoke_agent & get_agent tools
│   ├── config.ts  # Env loading, WO_* config
│   └── auth.ts    # IAM token, woFetch for Watson Orchestrate API
├── tests/
│   └── verify.ts  # Verification script (get_agent + invoke_agent)
├── examples/
│   ├── mcp.json           # Cursor / VS Code config
│   └── antigravity-mcp.json # Antigravity config
├── package.json
├── tsconfig.json
└── .env.example
```

## Legal

### License

Distributed under the [Apache License, Version 2.0](http://www.apache.org/licenses/LICENSE-2.0).

SPDX-License-Identifier: Apache-2.0
