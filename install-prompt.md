# Install Penfield

Copy the prompt below and paste it into your AI assistant to set up Penfield automatically.

---

## Meta Prompt

Copy everything inside the box below:

```text
Help me install Penfield, a persistent memory and knowledge graph system for AI assistants.

Penfield connects via MCP (Model Context Protocol). Detect which platform you're running on and follow the appropriate setup:

**If you're Claude Desktop / Claude.ai:**
Go to Settings > Connectors > Add Custom Connector.
- Name: Penfield
- Remote MCP server URL: https://mcp.penfield.app
Click Add, then Connect, then Authorize.

**If you're Claude Code:**
Run this command:
claude mcp add --transport http --scope user penfield https://mcp.penfield.app

**If you're any other MCP-compatible platform (Cursor, Windsurf, Gemini CLI, Perplexity, Antigravity, LM Studio, etc.):**
Add this to the platform's MCP configuration file:

{
  "mcpServers": {
    "Penfield": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://mcp.penfield.app/"]
    }
  }
}

Common config file locations:
- Windsurf: ~/.codeium/windsurf/mcp_config.json
- Cursor: ~/.cursor/mcp.json
- Gemini CLI: ~/.gemini/settings.json
- Antigravity: ~/.gemini/antigravity/mcp_config.json
- LM Studio: Program > Install > Edit mcp.json

After adding the config, restart the application. A browser window will open for authorization. Complete the authorization flow.

**To verify the connection works:** Type "Awaken" and Penfield should respond with a personalized briefing.

**Optional next steps:**
- Visit https://portal.penfield.app to customize your AI personality
- See the full documentation at https://docs.penfield.app

If anything goes wrong, check:
1. Node.js is installed (required for mcp-remote): node --version
2. The JSON syntax is correct (no trailing commas, matching braces)
3. You restarted the application after adding the config
4. Try authorizing through a different application first if auth doesn't trigger

Support: support@penfield.app
```

---

Full setup guides with screenshots:

- [Claude.ai Setup](onboarding/penfield-setup-guide.md)
- [Claude Code Setup](onboarding/claude-code-setup.md)
- [MCP Remote Setup (Cursor, Windsurf, etc.)](onboarding/mcp-remote-setup.md)
