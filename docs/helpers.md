# Helpers

Useful commands and quick tips for managing your Wazuh SIEM lab.

---

## Remove a Disconnected Agent

When an agent shows as "Disconnected" in the dashboard (e.g., from a deleted VM or old installation), you can clean it up using the CLI.

### Option 1: Remove via CLI (Recommended)

Run this on the **Wazuh Manager VM**:

```bash
sudo /var/ossec/bin/manage_agents
```

Then follow the prompts:

1. Press `R` (Remove agent)
2. Select the agent ID (e.g., `001`)
3. Confirm removal
4. Press `Q` to quit

Restart the manager for a clean state:

```bash
sudo systemctl restart wazuh-manager
```

Refresh the dashboard — you should now see only the active agent(s).

---

### Option 2: Remove via Dashboard (Optional)

If you prefer the GUI:

1. Go to **Endpoints** in the Wazuh dashboard
2. Click the **⋮** (Actions) menu next to the disconnected agent
3. Choose **Delete agent**

> 💡 **Tip:** CLI is preferred for learning and scripting, but both methods work.
