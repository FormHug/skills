---
name: setup-formhug-mcp
description: |
  Set up and connect to FormHug's remote MCP server with OAuth authentication.
  Use when the user wants to connect to FormHug MCP, set up FormHug integration,
  add FormHug as an MCP server, or troubleshoot FormHug MCP connection issues.
  Triggers on "setup formhug", "connect formhug", "add formhug mcp",
  "formhug mcp setup", "configure formhug".
---

# Set Up FormHug MCP Server

You are a setup assistant that helps users connect their AI agent or IDE to FormHug's remote MCP server at `https://formhug.ai/mcp`. FormHug's MCP server uses OAuth 2.1 authorization (per the [MCP Authorization spec](https://modelcontextprotocol.io/docs/tutorials/security/authorization)).

## Prerequisites

- An MCP-compatible agent or IDE (Claude Code, Cursor, Windsurf, Claude Desktop, etc.)
- A FormHug account (sign up at https://formhug.ai if needed)
- A web browser for the OAuth login flow

## MCP Server Details

| Property | Value |
|----------|-------|
| **URL** | `https://formhug.ai/mcp` |
| **Transport** | Streamable HTTP |
| **Auth** | OAuth 2.1 (authorization code + PKCE) |

## Workflow

### Step 1: Detect the user's agent

Ask the user which agent or IDE they are using if not obvious from context. The setup steps differ per tool.

### Step 2: Add the FormHug MCP server

Follow the instructions for the user's agent:

#### Claude Code (CLI)

```bash
claude mcp add --transport http formhug https://formhug.ai/mcp
```

**Scope options:**

| Scope | Flag | Effect |
|-------|------|--------|
| **User** (Recommended) | `--scope user` | Available across all your projects |
| **Project** | `--scope project` | Shared via `.mcp.json` in the project root |
| **Local** (default) | (no flag) | This project only, private to you |

#### Claude Desktop

Edit the Claude Desktop config file:

- **macOS:** `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Windows:** `%APPDATA%\Claude\claude_desktop_config.json`

Add the following under `"mcpServers"`:

```json
{
  "mcpServers": {
    "formhug": {
      "type": "url",
      "url": "https://formhug.ai/mcp"
    }
  }
}
```

Then restart Claude Desktop.

#### Cursor

Open Cursor Settings > MCP, click "+ Add new MCP server", and enter:

- **Name:** `formhug`
- **Type:** `url` (Streamable HTTP)
- **URL:** `https://formhug.ai/mcp`

Or add directly to `.cursor/mcp.json` in your project:

```json
{
  "mcpServers": {
    "formhug": {
      "type": "url",
      "url": "https://formhug.ai/mcp"
    }
  }
}
```

#### Windsurf

Add to `~/.codeium/windsurf/mcp_config.json`:

```json
{
  "mcpServers": {
    "formhug": {
      "type": "url",
      "url": "https://formhug.ai/mcp"
    }
  }
}
```

Then restart Windsurf.

#### Other MCP-compatible agents

For any agent that supports remote MCP servers via Streamable HTTP, add a server with:

- **URL:** `https://formhug.ai/mcp`
- **Transport:** Streamable HTTP (sometimes labeled "HTTP" or "url")

The agent should handle OAuth 2.1 discovery and authorization automatically per the MCP spec.

### Step 3: Authenticate

When the agent connects to the FormHug MCP server for the first time, it will:

1. Receive a `401 Unauthorized` response with OAuth metadata
2. Discover FormHug's authorization server endpoints
3. Open a browser window for login
4. The user logs in with their FormHug account and grants access
5. The agent receives and stores an access token

If the browser doesn't open automatically, look for a URL in the agent's output and open it manually.

#### Agent-assisted OAuth flow

Some agents (e.g., custom AI agents, headless environments, or agents without browser access) cannot open a browser automatically. In this case, the agent should handle the OAuth flow interactively:

1. The agent constructs the OAuth authorization URL (with PKCE challenge)
2. **Present the authorization URL to the user** and ask them to open it in their browser
3. The user logs in, grants access, and is redirected to the callback URL
4. **Ask the user to copy the full callback URL** (from the browser's address bar) and send it back to the agent
5. The agent extracts the authorization code from the callback URL and exchanges it for an access token

**Example agent prompt to the user:**

> Please open this link in your browser to authorize FormHug:
>
> `https://formhug.ai/oauth/authorize?client_id=...&redirect_uri=...&code_challenge=...`
>
> After you log in and approve access, your browser will redirect to a URL. Please copy the **full URL** from your browser's address bar and paste it here.

The agent then parses the callback URL to extract the `code` parameter and completes the token exchange.

### Step 4: Verify the connection

Confirm the server is connected and tools are available. Try a simple operation like listing forms to verify end-to-end connectivity.

## Removing the Server

#### Claude Code
```bash
claude mcp remove formhug
```

#### Other agents
Remove the `"formhug"` entry from the relevant config file and restart the agent.

## Troubleshooting

### Server not connected or tools not appearing
1. Verify the URL is exactly `https://formhug.ai/mcp`
2. Check the config file for JSON syntax errors
3. Remove and re-add the server entry
4. Restart the agent

### OAuth / Authentication errors
1. Remove and re-add the server to clear cached auth
2. Ensure the FormHug account is active and not locked
3. Check that the browser isn't blocking popups from the OAuth flow
4. If behind a corporate proxy, ensure `formhug.ai` is accessible

### Token expired errors
Most agents auto-refresh OAuth tokens. If errors persist:
1. Remove the server configuration
2. Re-add it and re-authenticate through the browser flow

## What FormHug MCP Provides

The available tools may change as FormHug evolves. Connect the server and check your agent's tool list for the latest. Below is a reference snapshot:

| Tool | Description |
|------|-------------|
| `list_forms` | List forms you own |
| `get_form` | Get form details and field definitions |
| `create_form` | Create a new form |
| `update_form` | Update form name/description |
| `delete_form` | Permanently delete a form |
| `add_fields` | Add fields to an existing form |
| `remove_field` | Remove a field from a form |
| `preview_form` | Get a readable markdown preview |
| `list_entries` | List form submissions |
| `get_entry` | Get a single submission |
| `submit_entry` | Submit a new entry to a form |
