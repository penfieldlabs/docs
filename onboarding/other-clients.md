# Other MCP Clients

Connect Penfield to any AI app or agent that supports remote MCP servers. Use this guide when your AI does not have its own setup page.

Most AI apps can add a remote MCP server. The name of the feature changes from product to product: **connector**, **custom connector**, **plugin**, **app**, **integration**, **tool** or **MCP server**. They all work the same way with Penfield. You give the app one URL and sign in.

## Prerequisites

- A Penfield account ([sign up here](https://portal.penfield.app/sign-up) if you haven't already)
- An AI app or agent that can connect to a remote MCP server

---

## Connection Details

| Setting | Value |
|---------|-------|
| **Name** | `Penfield` |
| **Server URL** | `https://mcp.penfield.app` |
| **Transport** | Streamable HTTP (some apps call it "HTTP" or "streaming HTTP") |
| **Authentication** | OAuth. You sign in to Penfield in your browser |
| **Client ID and secret** | Leave empty. The app registers itself with Penfield automatically |
| **Headers** | Leave empty. No API key is needed |

---

## Step 1: Find Where Your App Adds MCP Servers

Open your AI app's settings and look for connectors, plugins, apps, integrations, tools or MCP servers. Choose the option to add a **custom** or **remote** server.

Apps that are configured with a file, such as code editors and command-line agents, have an MCP section in their configuration file. See [Configuration Files](#configuration-files) below.

---

## Step 2: Add Penfield

Enter the values from [Connection Details](#connection-details). In most apps you only fill in two fields:

| Field | Value |
|-------|-------|
| **Name** | `Penfield` |
| **Server URL** | `https://mcp.penfield.app` |

If the app asks how to authenticate, choose **OAuth**. If it asks how to register the OAuth client, choose the automatic option (often called **dynamic client registration** or **register automatically**).

---

## Step 3: Sign In and Authorize

The app opens a Penfield page in your browser. Sign in to your Penfield account if asked, then click **Authorize**.

---

## Step 4: Activate Your Assistant

Start a new conversation and type:

```
Penfield Awaken
```

Penfield will load your personality configuration and respond with a personalized greeting. Start each new conversation this way.

Some apps need the connector turned on for each conversation, or selected by typing `@Penfield` before the message. If your AI does not respond to "Penfield Awaken", check that Penfield is enabled for the conversation, then ask it directly: "Call the awaken tool from Penfield."

When the app asks for permission to use a Penfield tool, allow it. Choosing the "always allow" option is up to you, of course, but it lets your assistant use its memory seamlessly, without asking each time.

---

## Configuration Files

Apps that read MCP servers from a JSON file take an entry like this. The exact key names vary by app, so check your app's documentation for remote (HTTP) MCP servers:

```json
{
  "mcpServers": {
    "penfield": {
      "type": "http",
      "url": "https://mcp.penfield.app"
    }
  }
}
```

If the app can only start local MCP servers, or does not support OAuth sign-in for remote servers, connect through `mcp-remote`, which handles the connection and the sign-in for the app:

```json
{
  "mcpServers": {
    "Penfield": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote",
        "https://mcp.penfield.app/"
      ]
    }
  }
}
```

See [MCP Remote Setup](mcp-remote-setup.md) for details and app-specific steps.

---

## Technical Details

For developers and for apps that ask for more than a URL.

| Item | Value |
|------|-------|
| MCP endpoint | `https://mcp.penfield.app` (the root path; there is no `/mcp` or `/sse` path) |
| Transport | Streamable HTTP. The older HTTP+SSE transport is not offered |
| MCP protocol versions | `2025-11-25`, `2025-06-18`, `2025-03-26`, `2024-11-05` |
| Capabilities | Tools (17) and prompts (`awakening_protocol`, `checkpoint_protocol`) |
| Authentication | OAuth 2.1, authorization code flow with PKCE (`S256`) |
| Client registration | Dynamic Client Registration (RFC 7591). A client must be registered before it starts the sign-in |
| Protected resource metadata | `https://mcp.penfield.app/.well-known/oauth-protected-resource` |
| Authorization server | `https://auth.penfield.app` (metadata at `https://auth.penfield.app/.well-known/oauth-authorization-server`) |
| Scopes | `read` and `write` are required. Add `offline_access` to receive a refresh token |
| Resource parameter | `https://mcp.penfield.app` |

A request without a valid token gets `401` with a `WWW-Authenticate` header that points to the protected resource metadata, which is how MCP clients discover the sign-in.

The tools are described in the [MCP Tool Reference](../mcp/mcp-integration.md). OAuth is described in [Authentication](../api/endpoints/authentication.md).

---

## Troubleshooting

| Problem | What to check |
|---------|---------------|
| The app reports `404` or "not found" | Use the bare URL `https://mcp.penfield.app`. Do not add `/mcp` or `/sse` |
| The app reports `401` or "authentication required" and no sign-in page opens | The app did not start the OAuth sign-in. Check that it supports OAuth for remote MCP servers. If it does not, use [mcp-remote](mcp-remote-setup.md) |
| You pasted an API key into a header field | Remove it. The MCP server does not accept API keys. API keys are for the [REST API](../api/README.md) |
| The app asks for a client ID and has no automatic option | Register a client yourself as shown in [Authentication](../api/endpoints/authentication.md#step-2-register-client-optional), using the redirect URL your app gives you, and enter the `client_id` it returns |
| The app only supports the SSE transport or only local servers | Use [mcp-remote](mcp-remote-setup.md) |
| The sign-in worked before and tools now fail | Remove the connection in the app and add it again |
| Your AI has no MCP support at all | Use the [REST API](../api/README.md). The [Install Prompt](../install-prompt.md) helps your AI work out the best way to connect |

---

## Support

If you encounter issues, contact [support@penfield.app](mailto:support@penfield.app). Tell us which AI app you are connecting and what it shows.
