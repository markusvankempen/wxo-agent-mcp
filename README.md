# WxO Agent MCP

Simple MCP (Model Context Protocol) server that **invokes a single Watson Orchestrate agent** remotely. The agent is defined once via environment variables or MCP config.

Use this when you want a lightweight MCP that only chats with one agent—no tool management, no agent listing, no flows. Just `invoke_agent(message)` and `get_agent()`.

**Full documentation:** [DOCUMENTATION.md](DOCUMENTATION.md)

### Architecture

```
┌──────────────────────────────────────────────────┐  stdio    ┌─────────────────────┐  HTTP   ┌──────────────────────┐
│  Cursor • VS Code • Langflow • Claude • etc.   │ ◄────────► │  wxo-agent-mcp     │ ──────► │  Watson Orchestrate  │
│  (MCP clients)                                   │  JSON-RPC  │  invoke_agent       │  REST   │  (agent + tools+LLM) │
└──────────────────────────────────────────────────┘           │  get_agent          │         └──────────────────────┘
                                                                 └─────────────────────┘
```

## Tools

| Tool | Description |
|------|-------------|
| **invoke_agent** | Send a message to the configured Watson Orchestrate agent. The agent responds using its tools and LLM. |
| **get_agent** | Get details of the configured agent (name, description, tools, instructions). |

## Configuration

Set these in `.env` or your MCP client config (e.g. Cursor `mcp.json`):

```env
WO_API_KEY=your-ibm-cloud-api-key
WO_INSTANCE_URL=https://your-instance-id.orchestrate.ibm.com
WO_AGENT_ID=your-agent-id
# Or WO_AGENT_IDs=id1,id2  (first is used)
```

## Quick Start

```bash
npm install
npm run build
cp .env.example .env
# Edit .env with your credentials and agent ID
WO_API_KEY=... WO_INSTANCE_URL=... WO_AGENT_ID=... node dist/index.js
```

### Verify

```bash
# Default question ("Hello, who are you?")
WO_API_KEY=... WO_INSTANCE_URL=... WO_AGENT_IDs=... npm run test:verify

# Custom question
npm run test:verify -- -ask "What is the weather in Amsterdam?"
```

Runs `get_agent` and `invoke_agent` to confirm connectivity.

### Test in VS Code

1. Open the `wxo-agent-mcp` folder in VS Code.
2. Run `npm run build`.
3. `.vscode/mcp.json` registers the MCP server as **wxo-agent** and loads `.env`.
4. Open Copilot Chat (Ctrl+Shift+I) and ask: *Use invoke_agent to ask: What is the weather in Amsterdam?*  
   For "what can you do", use *Use invoke_agent to ask: What can you do?* to avoid Copilot calling `get_agent` first.
5. Or run `npm run test:verify` from the terminal (Ctrl+`).

See [DOCUMENTATION.md](DOCUMENTATION.md#testing-locally-in-vs-code) for more prompts and setup.

**Question examples:** [Full list →](DOCUMENTATION.md#question-examples)

| invoke_agent | get_agent |
|--------------|-----------|
| What is the weather in Amsterdam? | Use get_agent to show agent details |
| Tell me a dad joke | What tools does my agent have? |
| What time is it in Tokyo? | |
| What can you help me with? | |

**Langflow:** Add an MCP Tools component, choose STDIO, set Command=`node`, Args=`["/path/to/dist/index.js"]`, and env vars. See [DOCUMENTATION.md](DOCUMENTATION.md#testing-in-langflow).

## MCP Client Configuration

### Cursor (`.cursor/mcp.json`)

```json
{
  "mcpServers": {
    "wxo-agent": {
      "command": "npx",
      "args": ["-y", "wxo-agent-mcp"],
      "env": {
        "WO_API_KEY": "your-api-key",
        "WO_INSTANCE_URL": "https://xxx.orchestrate.ibm.com",
        "WO_AGENT_ID": "your-agent-id"
      }
    }
  }
}
```

### VS Code Copilot (`.vscode/mcp.json`)

The server is named **wxo-agent**. Use `envFile` to load `.env`:

```json
{
  "servers": {
    "wxo-agent": {
      "type": "stdio",
      "command": "node",
      "args": ["${workspaceFolder}/dist/index.js"],
      "envFile": "${workspaceFolder}/.env"
    }
  }
}
```

For npx (after publishing), use `"command": "npx"`, `"args": ["-y", "wxo-agent-mcp"]`, and add `env` with your credentials.

### Antigravity

Same config as Cursor. Copy `examples/antigravity-mcp.json` or add to your MCP config:

```json
{
  "mcpServers": {
    "wxo-agent": {
      "command": "npx",
      "args": ["-y", "wxo-agent-mcp"],
      "env": {
        "WO_API_KEY": "your-api-key",
        "WO_INSTANCE_URL": "https://xxx.orchestrate.ibm.com",
        "WO_AGENT_ID": "your-agent-id"
      }
    }
  }
}
```

## vs wxo-builder-mcp-server

| | wxo-agent-mcp | wxo-builder-mcp-server |
|---|---------------|------------------------|
| Purpose | Invoke one agent | Full dev toolkit (tools, agents, connections, flows) |
| Agent | Single `WO_AGENT_ID` | Multiple agents, `WO_AGENT_IDs` |
| Tools | `invoke_agent`, `get_agent` | 30+ tools (list_skills, deploy_tool, etc.) |
| Use case | Chat with a specific agent | Build and manage Watson Orchestrate resources |

## License

Apache-2.0
