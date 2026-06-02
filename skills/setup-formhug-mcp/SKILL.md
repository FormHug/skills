---
name: setup-formhug-mcp
description: |
  Install and connect to FormHug's remote MCP server (https://formhug.ai/mcp).
  Use when the user asks to set up / connect / add / configure FormHug,
  or when FormHug MCP tools are needed but not yet available in this session.
  Picks the right auth method (OAuth or Personal Access Token) based on the
  current agent's environment, then writes the correct MCP config for that
  agent and verifies the connection with `get_me`.
---

# Set up FormHug MCP

You are configuring the **current agent** (the one reading this skill) to talk to FormHug's remote MCP server. Drive the whole process yourself; only involve the user for the small parts they must do (logging in, creating a token).

## Server facts

| Property      | Value                              |
| ------------- | ---------------------------------- |
| **URL**       | `https://formhug.ai/mcp`           |
| **Transport** | Streamable HTTP (only)             |
| **Auth**      | OAuth 2.1 **or** Personal Access Token (`Authorization: Bearer fh_...`) |

## Step 0 — Skip if already connected

If FormHug MCP tools (`get_me`, `list_forms`, …) are already available in your toolset, call `get_me` once. If it returns a user, tell the user "FormHug is already connected as `<email>`" and **stop**. Don't reinstall.

## Step 1 — Pick the auth method by agent identity

The choice is fixed by **what agent you are**, not by user preference:

| Agent you are                                                                | Auth method     | Why                                                                 |
| ---------------------------------------------------------------------------- | --------------- | ------------------------------------------------------------------- |
| Claude Code, Codex CLI, Cursor, Windsurf, Claude Desktop, VS Code (Copilot), Cline, Zed, any local IDE/CLI agent | **OAuth 2.1**   | Runs on the user's machine — the agent's MCP client opens a browser and handles OAuth end-to-end. |
| OpenClaw, Hermes, any remote / chat-only agent that cannot launch a browser on the user's machine | **PAT**         | No browser bridge. OAuth would force the user to paste the redirect URL back, which is unacceptable per project policy. |
| Genuinely unsure                                                              | Ask the user    | "Can you open a browser on the machine I'm running on, or am I connected to you only through chat?" Browser → OAuth. Chat-only → PAT. |

**Do not** try to drive OAuth manually (fetching metadata, building authorization URLs, asking the user to paste the redirect URL). If OAuth isn't viable for your environment, use PAT.

---

## OAuth path (local agents)

Add the server with the agent-specific command/config below. The agent's MCP client takes care of OAuth discovery, PKCE, browser launch, and token storage. After adding, the user will be prompted by the agent (not by you) to log in.

### Claude Code

```bash
claude mcp add --transport http formhug https://formhug.ai/mcp --scope user
```

`--scope user` makes it available across all projects. Use `--scope project` to share via `.mcp.json`, or omit for project-local. After adding, Claude Code will prompt the user to authorize in the browser the first time a FormHug tool is called.

> ⚠️ Claude Code loads MCP servers at startup. If the FormHug tools don't appear in your current session, tell the user they likely need to **restart Claude Code** for the new server to take effect.

### Codex CLI

Add to `~/.codex/config.toml`:

```toml
[mcp_servers.formhug]
url = "https://formhug.ai/mcp"
```

Codex CLI handles OAuth automatically when no `bearer_token_env_var` is set.

> ⚠️ Codex CLI starts MCP servers only at launch. Remind the user to **restart Codex CLI** after editing the config for the server to be picked up.

### Cursor

Add to `~/.cursor/mcp.json` (or `.cursor/mcp.json` in the project for project scope):

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

> ⚠️ Cursor only reads the MCP config on startup. Remind the user to **restart Cursor** (or hit the refresh button in MCP settings) for the new server to load.

### Windsurf

Add to `~/.codeium/windsurf/mcp_config.json`:

```json
{
  "mcpServers": {
    "formhug": {
      "serverUrl": "https://formhug.ai/mcp"
    }
  }
}
```

Then click **Refresh** in the Cascade panel.

### Claude Desktop

Add to `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS) or `%APPDATA%\Claude\claude_desktop_config.json` (Windows):

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

Then **fully quit and relaunch** Claude Desktop.

### Other local MCP clients

Use the client's standard "remote MCP server (Streamable HTTP)" entry with URL `https://formhug.ai/mcp` and no auth header — the client will discover OAuth from the 401 response per the MCP authorization spec.

---

## PAT path (remote / chat-only agents)

For agents that cannot drive a browser-based OAuth flow.

**Critical rule: never ask the user to send the token back through chat.** The token is a long-lived credential equivalent to a password; if it traverses the chat channel it ends up in model provider logs, conversation history, possibly training data, and any future screenshot/share of this conversation. The flow below is designed so the token goes **directly from FormHug to the user's MCP config**, bypassing you entirely.

You give the user (a) instructions to mint the token, and (b) a config snippet with a placeholder. The user fills the token in themselves and tells you when done. Then you verify.

### Step P1 — Send the user the full instructions in one message

Compose **one message** to the user containing both the token-creation steps and the config snippet they need to paste, with a `<PASTE YOUR TOKEN HERE>` placeholder where the token goes. The user should never need to send the token to you.

Template (adapt the config snippet to the user's agent — see below):

> To connect FormHug, please do these steps on your end:
>
> **1. Create a Personal Access Token**
>
> 1. Open <https://formhug.ai/profile/personal_access_token>
> 2. Click **Create Token**, give it a name (e.g. the name of this agent), pick the scopes you want me to have, click **OK**
> 3. Copy the token — it starts with `fh_` and is shown **only once**. Save it in a password manager.
>
> **2. Add this to your MCP config**, replacing the placeholder with the token you just copied:
>
> ```json
> { /* the snippet for your specific agent, see below */ }
> ```
>
> Config file location: `<path for the user's agent>`
>
> **3.** Let me know once you've saved the file, and I'll verify the connection.

Then wait for the user. If they accidentally send the token anyway, tell them to **revoke it immediately** at the same page and create a fresh one — once it has appeared in chat it must be considered leaked.

### Step P2 — Per-agent config snippets

#### OpenClaw

Config file: `~/.openclaw/openclaw.json` (the OpenClaw Gateway hot-reloads it; no restart needed).

```json
{
  "mcpServers": {
    "formhug": {
      "type": "url",
      "url": "https://formhug.ai/mcp",
      "headers": {
        "Authorization": "Bearer <PASTE YOUR TOKEN HERE>"
      }
    }
  }
}
```

#### Generic remote / chat agent

If you're some other remote agent, give the user a snippet in your platform's documented MCP config format with:

- `url`: `https://formhug.ai/mcp`
- transport: Streamable HTTP
- header: `Authorization: Bearer <PASTE YOUR TOKEN HERE>`

If the platform supports secrets / environment variables, prefer referencing the token by env var name rather than inline (e.g., `${env:FORMHUG_PAT}`) and instruct the user to add the secret through the platform's secrets UI instead of editing the JSON directly.

---

## Step 2 — Verify

Call `get_me`. On success, tell the user "FormHug connected as `<email>`". On failure, surface the error verbatim — FormHug's error responses are self-explanatory (401 = bad/missing token, 403 = scope missing, etc.). Common fixes:

- 401 → token wrong, expired, or revoked → ask the user to mint a new one (PAT) or trigger the agent's MCP "re-authenticate" command (OAuth)
- Tool not found → the config wasn't loaded; restart/reload as required by the agent (see per-agent notes above)
- Network error → check `https://formhug.ai/mcp` is reachable from the agent's host

