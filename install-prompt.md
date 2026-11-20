# Install Penfield

Copy the prompt below and paste it into your AI assistant. It tells your AI what Penfield is, helps it work out the best way to connect from wherever it is running, and walks you through the setup.

---

## Meta Prompt

Copy everything inside the box below:

```text
Help me set up Penfield. Read all of this before you answer, then work out the best way to connect Penfield to the environment you are running in and guide me through it.

== WHAT PENFIELD IS ==

Penfield is persistent memory for AI assistants. It gives you long-term memory that survives across conversations:
- Memories: individual pieces of knowledge (facts, insights, decisions, corrections, tasks), up to 10,000 characters each, with tags.
- Relationships: typed links between memories (supports, supersedes, causes, references, and 20 more) that form a knowledge graph.
- Hybrid search: keyword (BM25), semantic (vector) and graph search combined.
- Documents: files the user uploads, split into chunks and searchable alongside memories.
- Artifacts: file storage with folders, for code and long content. Artifacts are read by path and are not searched.
- Contexts: named checkpoints that hand work off to a later session or another agent.
- Personality: a persona and custom instructions the user sets in the Penfield portal. You load them by calling "awaken".

Accounts and settings: https://portal.penfield.app (sign up at https://portal.penfield.app/sign-up)
Documentation: https://docs.penfield.app
Overview written for AI models: https://docs.penfield.app/llms.txt

== THE THREE WAYS TO CONNECT ==

1. REMOTE MCP SERVER. Use this whenever the app supports it.
   - Server URL: https://mcp.penfield.app
   - Transport: streamable HTTP
   - Sign-in: OAuth in the user's browser. The app registers itself automatically (dynamic client registration). No API key and no custom headers.
   - Apps name this feature differently: connector, custom connector, plugin, app, integration, MCP server. It is the same thing.

2. MCP-REMOTE BRIDGE. Use this when the app only runs local MCP servers from a config file, or cannot do the OAuth sign-in for a remote server. Needs Node.js.
   {
     "mcpServers": {
       "Penfield": {
         "command": "npx",
         "args": ["-y", "mcp-remote", "https://mcp.penfield.app/"]
       }
     }
   }
   A browser window opens for the sign-in the first time.

3. REST API. Use this for scripts, backends and agents that have no MCP support but can make HTTP requests.
   - Base URL: https://api.penfield.app/api/v2
   - API key: the user creates one in the portal (API Keys). Exchange it for an access token:
       POST https://api.penfield.app/api/v2/auth/token   with header   Authorization: Bearer <API key>
     The response has an access_token (valid 24 hours) and a refresh_token. Send the access token as "Authorization: Bearer <access token>" on every other call. An API key cannot be used directly on those calls, and it cannot be used with the MCP server.
   - Device code flow (OAuth, RFC 8628): for command-line tools and headless agents where the user approves in a browser on another device. Discover the endpoints at https://api.penfield.app/.well-known/oauth-authorization-server
   - Authorization code with PKCE: for apps that can open a browser and receive a redirect.
   - Full reference: https://docs.penfield.app/api/

== STEP 1: WORK OUT WHAT YOU ARE RUNNING IN ==

Decide which of these describes you. If you are not sure, ask me which app or tool I am using. Do not guess.
a) A chat app where the user can add a remote MCP server, custom connector or plugin  -> use way 1.
b) A coding agent, IDE or desktop app configured through an MCP config file          -> use way 1 if it supports remote HTTP servers with OAuth, otherwise way 2.
c) A script, backend or custom agent with no MCP support                             -> use way 3.
d) None of the above (no tools, no MCP, no HTTP access)                              -> tell me plainly that you cannot connect from here, and suggest one of the apps below.

If you can browse the web, check https://docs.penfield.app for a setup guide for my app before giving instructions, because app menus change.

== STEP 2: MAKE SURE I HAVE A PENFIELD ACCOUNT ==

Ask whether I already have one. If not, send me to https://portal.penfield.app/sign-up first.

== STEP 3: CONNECT ==

Most steps happen in the app's settings, which you usually cannot change yourself. Give me exact, numbered instructions and wait for me to confirm each stage. If you can run commands or edit files, offer to do it and show me what you will change first.

Known apps:
- Claude (claude.ai, desktop, mobile): Plugins (left sidebar) > Add > "Add custom connector". Name: Penfield. Server address: https://mcp.penfield.app. Continue. Keep the detected settings ("Sign in now" and "Register automatically"), scroll down, click Add. Then sign in to Penfield and click Authorize.
- Claude Code: run   claude mcp add --transport http --scope user penfield https://mcp.penfield.app   then type /mcp, select penfield and choose Authenticate.
- ChatGPT (web): Plugins (left sidebar) > Add > "Add custom MCP server". Name: Penfield. Server URL: https://mcp.penfield.app. Authentication: OAuth. Scroll down, tick "I understand and want to continue", click "Create as a plugin", then "Continue to Penfield" and Authorize. In a chat, type @Penfield.
- Grok: Settings > Connect Apps > scroll to the bottom > "Add a custom connector". Name: Penfield. Server URL: https://mcp.penfield.app. Leave headers empty. Add MCP, then click Authorize. In a chat, type @Penfield.
- Perplexity, Manus: add a custom connector with the same name and URL. Guides are at https://docs.penfield.app/onboarding/
- Cursor, Windsurf, Gemini CLI, Antigravity, LM Studio and similar: add the mcp-remote block from way 2 to the app's MCP config file, then restart the app.
    Windsurf: ~/.codeium/windsurf/mcp_config.json
    Cursor: ~/.cursor/mcp.json
    Gemini CLI: ~/.gemini/settings.json
    Antigravity: ~/.gemini/antigravity/mcp_config.json
    LM Studio: Program > Install > Edit mcp.json
- Any other app with MCP support: add a remote MCP server with the URL https://mcp.penfield.app, choose OAuth if asked, leave client ID, secret and headers empty. See https://docs.penfield.app/onboarding/other-clients/

== STEP 4: CHECK THAT IT WORKS ==

Ask me to type "Penfield Awaken" in a new conversation (in apps with mentions: "@Penfield awaken"). You should then be able to call the Penfield tool named awaken, which returns a briefing with my persona and custom instructions. If you can see Penfield tools such as awaken, store, recall and connect, the connection works.

If it does not work:
- "Not found" or 404: the URL must be exactly https://mcp.penfield.app with no /mcp or /sse at the end.
- 401 or "authentication required" with no sign-in page: the app did not start the OAuth sign-in. Use mcp-remote (way 2).
- An API key was pasted into a header: remove it. The MCP server does not accept API keys.
- mcp-remote problems: check that Node.js is installed (node --version), that the JSON is valid, and that the app was restarted.
- Still stuck: https://docs.penfield.app/onboarding/reconnect-penfield/ or support@penfield.app

== STEP 5: USE PENFIELD WELL ==

Once connected:
- Call awaken first in every new conversation, and follow the briefing it returns.
- Before answering something that may depend on earlier work, call recall.
- When something is worth keeping (a decision, a fact about me or my projects, a correction), call store. Write complete thoughts with context, and add tags.
- After storing, link related memories with connect. The links are what make later recall good.
- To correct a memory, call update_memory instead of storing a duplicate.
- At the end of a work session or before a handoff, call save_context with a narrative of where things stand.
- For code and long documents, use save_artifact. Artifacts are not searched, so also store a short memory that says what the file is and gives its exact path.
- Suggest that I set my persona and custom instructions at https://portal.penfield.app/personality

Rules for you while helping me:
- Never ask me to paste a password into this chat.
- Treat an API key as a secret. Keep it in an environment variable or a secret store, never in code, in a memory or in a file that gets shared.
- Say so when you are unsure about my app's menus, and point me to the setup guide instead of inventing steps.

Support: support@penfield.app
```

---

## Setup Guides

Full setup guides with screenshots:

- [ChatGPT Setup](onboarding/chatgpt-setup.md)
- [Claude Setup](onboarding/claude-setup.md)
- [Claude Code Setup](onboarding/claude-code-setup.md)
- [Grok Setup](onboarding/grok-setup.md)
- [Manus Setup](onboarding/manus-setup.md)
- [MCP Remote Setup (Cursor, Windsurf, etc.)](onboarding/mcp-remote-setup.md)
- [Perplexity Setup](onboarding/perplexity-setup.md)
- [Other MCP Clients](onboarding/other-clients.md)

For AI agents and LLMs there is also a plain-text overview of Penfield at [llms.txt](https://docs.penfield.app/llms.txt).
