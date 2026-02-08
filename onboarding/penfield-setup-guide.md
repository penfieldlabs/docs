# Penfield Setup Guide

Connect Penfield to Claude.ai and configure your personalized AI assistant.

## Prerequisites

- A valid Penfield invitation (check your email) or existing Penfield account
- A Claude.ai account with an active subscription
- A supported browser (Chrome, Firefox, or Brave recommended)

---

## Step 1: Sign Up for a Free Trial

Go to [https://portal.penfield.app/sign-up](https://portal.penfield.app/sign-up).

---

## Step 2: Create Your Penfield Account

If prompted, complete the Cloudflare verification by clicking the checkbox.

![Cloudflare verification](./img/setup-02-captcha.png)

Set a password, or select **Continue with Google** if your invitation was sent to a Gmail or Google Workspace address. Review and accept the Terms of Service, then click **Continue**.

![Account creation screen](./img/setup-03-create-account.png)

---

## Step 3: Set Up Claude.ai

If you don't already have a Claude account, create one at [claude.ai](https://claude.ai).

Sign in to Claude.ai. A paid Claude subscription (Pro or Team) is required to use custom connectors.

> **Note:** Safari may have compatibility issues. We recommend using Chrome, Firefox, or Brave.

---

## Step 4: Open Claude Settings

Click your profile icon in the bottom-left corner, then select **Settings**.

![Profile menu location](./img/setup-04-profile-icon.png)

![Settings option in menu](./img/setup-05-settings-menu.png)

---

## Step 5: Configure Capabilities (Recommended)

Navigate to **Settings > Capabilities**.

For optimal performance with Penfield, consider disabling Claude's built-in memory features. Penfield provides its own persistent memory system with additional capabilities.

> **Tip:** Claude Projects work well alongside Penfield. You can also use the [Penfield Document Portal](https://portal.penfield.app/documents) to import unlimited documents into your personal knowledge base.

![Capabilities settings](./img/setup-06-capabilities.png)

---

## Step 6: Configure Privacy Settings (Optional)

Navigate to **Settings > Privacy**.

Adjust these settings based on your preferences:

- **Help improve Claude** — Disable if you prefer your conversations not be used for model training
- **Location metadata** — Disable to limit location information shared with Claude

![Privacy settings](./img/setup-07-privacy.png)

---

## Step 7: Add the Penfield Connector

Navigate to **Settings > Connectors**, then click **Add Custom Connector**.

![Connectors settings page](./img/setup-08-connectors.png)

In the dialog, enter the following:

| Field | Value |
|-------|-------|
| **Name** | `Penfield` |
| **Remote MCP server URL** | `https://mcp.penfield.app` |

Click **Add**.

![Empty connector dialog](./img/setup-09-connector-dialog-empty.png)

![Completed connector dialog](./img/setup-10-connector-dialog-filled.png)

---

## Step 8: Connect and Authorize the Connection

Click **Connect** next to Penfield.
![Authorization prompt](./img/setup-11-connect.png)

Click **Authorize** to authorize Penfield for your Claude account.
![Authorization prompt](./img/setup-12-authorize.png)

---

## Step 9: Configure Your AI Personality

Visit the [Penfield Personality Portal](https://portal.penfield.app/personality) to customize your assistant's behavior, voice, and preferences.

![Personality Portal - selecting a persona](./img/setup-13-personality-portal.png)

You can choose from a standard persona and add custom instructions to personalize your assistant's behavior. Consider creating your own custom instructions or using [example custom instructions](example-custom-instructions.md)

![Personality Portal - custom instructions](./img/setup-14-personality-custom.png)

---

## Step 10: Activate Your Assistant

Start a new conversation in Claude and type:

```
Awaken
```

![Typing Awaken in Claude](./img/setup-15-awaken-input.png)

Penfield will load your personality configuration and respond with a personalized greeting.

![Awaken response showing personality loading](./img/setup-16-awaken-response.png)

---

## Step 11: Enable Penfield Tools

When Claude requests permission to use Penfield tools, select **Always allow** to enable seamless integration.

![Tool permission prompt](./img/setup-17-allow-tools.png)

You can also manage tool permissions in **Settings > Connectors > Penfield**.

![Connector settings page](./img/setup-18-connector-settings.png)

![Tool permissions configuration](./img/setup-19-tool-permissions.png)

---

## Usage Tips

- **Start each new conversation with "Awaken"** to load your Penfield configuration
- **Extended thinking mode** is recommended for most tasks
- **Custom styles** — Ask your Penfield assistant to develop, save, and recall writing styles

![Extended thinking option](./img/setup-20-extended-thinking.png)

> **Note:** The "Use style" and "Research" features in Claude may affect your assistant's personality. Feel free to use them, but be aware that your agent's personality may be affected. You can restore the original personality by disabling "Use style" and "Research". 

---

## Troubleshooting

If you encounter issues, see the [Reconnecting to Penfield](reconnect-penfield.md) guide.

For additional support, contact [support@penfield.app](mailto:support@penfield.app).
