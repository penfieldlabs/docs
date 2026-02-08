# Troubleshooting Penfield

Use this guide to diagnose and resolve connection issues with Penfield.

---

## Step 1: Verify Penfield Is Connected

Open the chat input menu in Claude and check that **Penfield** appears with the toggle enabled.

![Penfield connected in chat menu](./img/reconnect-01-penfield-connected.png)

If Penfield appears and is enabled, your connection is working. Try typing "Awaken" in a new chat to test.

---

## Step 2: Reconnect via Manage Connectors

If Penfield is missing from the menu, shows as unavailable, is disabled but cannot be re-enabled or otherwise continues to malfunction, click **Manage connectors**.

![Penfield missing - click Manage connectors](./img/reconnect-02-penfield-missing.png)

In **Settings > Connectors**, locate Penfield, if Penfield is disconnected, click **Connect**.

![Connectors page showing Penfield disconnected](./img/reconnect-03-connectors-disconnected.png)

When prompted, click **Authorize** to reconnect your Penfield account.

![Authorization prompt](./img/reconnect-04-authorize.png)

---

## Step 3: Disconnect and Reconnect

If Penfield appears connected but isn't functioning properly, try disconnecting and reconnecting:

1. Go to **Settings > Connectors**
2. Click the menu button (**⋯**) next to Penfield
3. Select **Disconnect**

![Disconnect option in menu](./img/reconnect-05-disconnect-menu.png)

4. After disconnecting, click **Connect** to re-establish the connection

![Reconnect Penfield](./img/reconnect-06-reconnect.png)

---

## Step 4: Remove and Reinstall

If reconnecting doesn't resolve the issue, remove and reinstall the Penfield connector:

### Remove the existing connector

1. Go to **Settings > Connectors**
2. Click the menu button (**⋯**) next to Penfield
3. Select **Disconnect**
4. Select **Remove**

![Select Disconnect from the menu](./img/reconnect-05-disconnect-menu.png)

![Select Remove from the menu](./img/reconnect-07-remove.png)

### Add the connector again

5. Click **Add custom connector**

![Click Add custom connector](./img/reconnect-08-connectors.png)

6. In the dialog, enter the connector details:

![Empty Add custom connector dialog](./img/reconnect-09-connector-dialog-empty.png)

| Field | Value |
|-------|-------|
| **Name** | `Penfield` |
| **Remote MCP server URL** | `https://mcp.penfield.app` |

7. Click **Add**

![Filled connector dialog with Penfield details](./img/reconnect-10-connector-dialog-filled.png)

### Connect and authorize

8. Click **Connect** next to the newly added Penfield connector

![Click Connect next to Penfield](./img/reconnect-11-connect.png)

9. Click **Authorize** to grant Penfield access to your workspace

![Click Authorize to complete setup](./img/reconnect-12-authorize.png)

---

## Still Having Issues?

Contact [support@penfield.app](mailto:support@penfield.app) for additional assistance.
