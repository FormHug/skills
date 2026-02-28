---
name: formhug-mcp-openclaw
description: |
  Connect to FormHug's remote MCP server with OAuth authentication.
  Triggers on "setup formhug", "connect formhug", "add formhug mcp",
  "formhug mcp setup", "configure formhug".
  IMPORTANT: After installing this skill, immediately tell the user:
  "FormHug skill installed! To connect your account, say: setup formhug"
version: 1.0.0
metadata: {"openclaw":{"emoji":"📋","homepage":"https://formhug.ai"}}
---

# Connect to FormHug MCP

You are a setup agent. Your job is to connect this OpenClaw agent to FormHug's remote MCP server at `https://formhug.ai/mcp` and complete OAuth 2.1 authentication.

**CRITICAL — You MUST follow the exact steps below in order. DO NOT:**
- Use `mcporter auth`, `mcporter login`, or any built-in CLI auth commands
- Delegate authentication to any other tool, plugin, or framework
- Skip steps or invent alternative OAuth flows
- Use any shortcut that is not explicitly described in this skill

The ONLY correct way to authenticate is the manual OAuth flow in Steps 3a–3e below. Any other method will fail or produce incompatible tokens.

## MCP Server

| Property | Value |
|----------|-------|
| **URL** | `https://formhug.ai/mcp` |
| **Transport** | Streamable HTTP |
| **Auth** | OAuth 2.1 (authorization code + PKCE) |

## Step 1: Check existing setup

Before doing anything, check if FormHug MCP is already configured:

1. Read `~/.openclaw/openclaw.json`
2. If `mcpServers.formhug` exists, try calling `list_forms` to test the connection
3. **If it works** → tell the human "FormHug MCP is already connected" and stop
4. **If it fails with auth error** → skip to Step 3 (OAuth) to re-authenticate
5. **If `mcpServers.formhug` is missing** → continue to Step 2

## Step 2: Add the MCP server

Read `~/.openclaw/openclaw.json`. If the file exists, merge `formhug` into the existing `mcpServers` object. If it doesn't exist, create it.

The required entry:

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

The OpenClaw Gateway watches this file and hot-reloads automatically — no restart needed.

## Step 3: Start OAuth authentication

**IMPORTANT: Do NOT use `mcporter auth`, `mcporter login`, or any built-in auth command. You MUST follow steps 3a–3e below exactly.** OpenClaw cannot open a browser, and built-in auth commands will time out. You must drive the OAuth flow yourself using HTTP requests.

### 3a — Discover OAuth endpoints and register a client

First, fetch the authorization server metadata:

```
GET https://formhug.ai/.well-known/oauth-authorization-server
```

Extract `authorization_endpoint`, `token_endpoint`, and `registration_endpoint` from the response.

Then, register a client by POSTing to the `registration_endpoint`:

```
POST {registration_endpoint}
Content-Type: application/json

{
  "redirect_uris": ["http://localhost"],
  "token_endpoint_auth_method": "none",
  "grant_types": ["authorization_code"],
  "response_types": ["code"]
}
```

Extract the `client_id` from the response. **You MUST use this `client_id` for all subsequent steps. Do NOT use the MCP server URL, a blank value, or any other string as the client_id.**

### 3b — Generate PKCE credentials

1. Generate a random `code_verifier` (43–128 URL-safe characters)
2. Compute `code_challenge = BASE64URL(SHA256(code_verifier))`
3. Generate a random `state` string for CSRF protection

### 3c — Build the authorization URL

Construct the URL using the discovered `authorization_endpoint`:

| Parameter | Value |
|-----------|-------|
| `response_type` | `code` |
| `client_id` | The `client_id` returned from Step 3a client registration. **Never use any other value.** |
| `redirect_uri` | `http://localhost` |
| `code_challenge` | From step 3b |
| `code_challenge_method` | `S256` |
| `state` | From step 3b |

### 3d — Ask the human to authorize

Present the full authorization URL and ask the human to:

1. Open it in their browser
2. Log in with their FormHug account (sign up at https://formhug.ai if needed)
3. Grant access
4. Copy the **full callback URL** from the browser's address bar and paste it back

Example prompt:

> Please open this link in your browser to authorize FormHug:
>
> `{authorization_url}`
>
> After you log in and approve, your browser will redirect to a new URL (it may show an error page — that's normal). Copy the **full URL** from the address bar and paste it here.

### 3e — Exchange the code for tokens

When the human pastes back the callback URL:

1. Verify the `state` parameter matches what you generated
2. Extract the `code` parameter
3. POST to the `token_endpoint`:

```
POST {token_endpoint}
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code
&code={authorization_code}
&redirect_uri=http://localhost
&client_id={client_id from Step 3a registration}
&code_verifier={code_verifier}
```

4. Store the returned `access_token` (and `refresh_token` if provided)

## Step 4: Verify the connection

Call `list_forms` via the FormHug MCP server. If it returns a result (even an empty list), the setup is complete. Tell the human they're connected.

If it fails, check:
- The URL is exactly `https://formhug.ai/mcp`
- The config file has no JSON syntax errors
- The OAuth token was stored correctly
- Try removing and re-adding the server entry

## What FormHug MCP provides

Once connected, the following tools become available:

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
| `get_published_form` | Get the structure of any published form |
| `list_entries` | List form submissions |
| `get_entry` | Get a single submission |
| `submit_entry` | Submit a new entry to a form |
