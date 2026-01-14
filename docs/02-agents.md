## Adding a Wazuh Agent (Endpoint)

This section documents how to deploy and enroll a Wazuh agent in a lab environment.  
The agent represents a monitored endpoint and runs on a separate Ubuntu VM, sending telemetry to the Wazuh manager.

This guide assumes the Wazuh manager, indexer, and dashboard are already installed and running (see `01-setup.md`).

## Architecture Overview

Wazuh follows a manager–agent model.

### Manager (Already Installed)

The manager is responsible for:

- Receiving logs and events from agents  
- Running detection rules and correlation  
- Storing data in the indexer  
- Providing the dashboard interface

### Agent (This Section)

The agent is a lightweight endpoint component that:

- Collects system logs and security events  
- Performs file integrity monitoring (FIM)  
- Sends data securely to the manager  
- Appears as a registered endpoint in the dashboard

⚠️ **Important**  
The agent must be installed on a separate VM.  
Installing an agent on the manager defeats the purpose of endpoint monitoring.

## Lab Requirements

### Virtual Machines

- **1 × Wazuh Manager VM** (from `01-setup.md`)  
- **1 × Agent (Client) VM**  
  Ubuntu Server or Desktop recommended

### Hardware Notes

**Manager VM**

- Minimum: **4 GB RAM** (often unstable)  
- Recommended: **8 GB RAM** for reliable startup and agent enrollment

**Agent VM**

- **1–2 GB RAM** is sufficient

### Network

- Both VMs must be able to communicate  
- Same NAT / host-only network is sufficient  
- IP-based communication is fine (no DNS required)

### Firewall Requirements (Manager VM)

If UFW is enabled on the manager, the following ports must be open before enrolling agents:

```bash
# Agent enrollment
sudo ufw allow 1515/tcp

# Agent communication
sudo ufw allow 1514/tcp
sudo ufw allow 1514/udp

sudo ufw reload
```

Without TCP/1515, agents will fail to enroll even if all services are running.

## Agent State in the Dashboard

### Before Deployment

When no agents are registered, the **Endpoints** page shows an empty state prompting deployment.

### After Successful Enrollment

Once an agent connects successfully, it appears as **Active**, with OS, IP, and version populated.

## Deploying the Agent (Dashboard-Guided Flow)

The Wazuh Dashboard provides a guided workflow that generates the correct installation commands for the agent OS.

The dashboard does **not** install the agent automatically.  
All commands must be run on the **agent VM**.

### Step 1: Start Agent Deployment

- Open the Wazuh Dashboard  
- Navigate to **Endpoints**  
- Click **Deploy new agent**

### Step 2: Select Platform and Package

Choose the operating system and architecture of the agent VM.

For Ubuntu labs:

- **Linux**  
- **DEB amd64**

The dashboard will tailor commands to this selection.

### Step 3: Server Address

Enter the Wazuh manager IP address:

- Use the private IP (e.g. `192.168.65.x`)  
- FQDN is optional for labs

This address is used by the agent to communicate with the manager.

### Step 4: Install the Agent (Agent VM)

Run the generated installation command on the agent VM.

Example for Ubuntu:

```bash
wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.14.1-1_amd64.deb
```

Install and register the agent:

```bash
sudo WAZUH_MANAGER="YOUR_MANAGER_IP" \
     WAZUH_AGENT_NAME="ubuntu-client-01" \
     dpkg -i wazuh-agent_4.14.1-1_amd64.deb
```

### Step 5: Start the Agent Service

After installation, enable and start the agent:

```bash
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```

Verify:

```bash
sudo systemctl status wazuh-agent
```

## Agent Enrollment (Required)

Installing the agent package alone is not sufficient.  
Each agent must authenticate with the manager.

Run on the **agent VM**:

```bash
sudo /var/ossec/bin/agent-auth -m YOUR_MANAGER_IP
```

Expected output:

```text
INFO: Authorization successful.
```

Restart the agent:

```bash
sudo systemctl restart wazuh-agent
```

## Verification

### On the Manager VM

List registered agents:

```bash
sudo /var/ossec/bin/agent_control -lc
```

Expected output:

```text
ID: 001, Name: ubuntu-client-01, Active
```

### In the Dashboard

- Go to **Endpoints**  
- Confirm the agent status is **Active**

## Common Pitfalls

- **Agent service running ≠ agent enrolled**  
- **UFW silently blocking TCP/1515**  
- **Manager running on low RAM delaying enrollment**

If the agent does not appear:

- Confirm manager services are running  
- Check port **1515** is listening  
- Review agent logs:

```bash
sudo tail -f /var/ossec/logs/ossec.log
```