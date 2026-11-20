# Grok Setup

Connect Penfield to Grok as a custom connector to enable persistent memory and personality features in your Grok conversations.

## Prerequisites

- A Penfield account ([sign up here](https://portal.penfield.app/sign-up) if you haven't already)
- A Grok account at [grok.com](https://grok.com)

---

## Step 1: Open Grok Settings

Click your name in the bottom-left corner of Grok, then click **Settings**.

![Click your name then Settings](./img/grok-01-open-settings.png)

---

## Step 2: Open Connect Apps

Click **Connect Apps** in the left sidebar, then scroll to the bottom of the list.

![Connect Apps page](./img/grok-02-connect-apps.png)

---

## Step 3: Add a Custom Connector

At the bottom of the list, click **Add a custom connector**.

![Add a custom connector button](./img/grok-03-add-custom-connector.png)

---

## Step 4: Add the Penfield MCP Server

Fill in the Add MCP Server form:

| Field | Value |
|-------|-------|
| **Name** | `Penfield` |
| **Server URL** | `https://mcp.penfield.app` |
| **Headers** | Leave empty |

Click **Add MCP**.

![Add MCP Server form with Penfield details](./img/grok-04-add-mcp-server.png)

---

## Step 5: Authorize the Connection

A Penfield page opens. Sign in to your Penfield account if asked, then click **Authorize** to connect your Penfield workspace to Grok.

![Penfield authorization prompt](./img/grok-05-authorize.png)

---

## Step 6: Activate Your Assistant

Start a new chat in Grok and type:

```
@Penfield awaken
```

Typing `@Penfield` selects the Penfield connector for the message.

![Typing @Penfield awaken in Grok](./img/grok-06-awaken.png)

Penfield will load your personality configuration and respond with a personalized greeting. Start each new chat this way.

---

## Support

If you encounter issues, contact [support@penfield.app](mailto:support@penfield.app).
