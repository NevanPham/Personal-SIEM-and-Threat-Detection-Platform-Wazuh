This section documents how to add a new agent (endpoint) to a Wazuh SIEM deployment.  
In this lab, the agent runs on a separate Ubuntu VM that simulates a client machine being monitored by the Wazuh manager.

## Overview

Wazuh follows a manager–agent architecture:

**Manager VM**  
Central component responsible for:

- Receiving events from agents  
- Correlation, rules, and alerting  
- Storing data in the indexer  
- Providing the dashboard UI

**Agent VM (Client)**  
Lightweight endpoint that:

- Collects logs, system events, file integrity data  
- Sends telemetry to the manager  
- Appears as a registered agent in the dashboard

⚠️ **Important**  
The agent must be installed on a separate machine.  
Running the agent on the manager VM defeats the purpose of endpoint monitoring.

## Lab Requirements

### Virtual Machines

- **1 × Manager VM**
- **1 × Agent (Client) VM**  
  Ubuntu Server/Desktop is recommended for consistency.

### Hardware (Strong Recommendation)

- **At least 8 GB RAM on the Manager VM**
  - Wazuh Indexer (OpenSearch) is memory-intensive  
  - 4 GB is the absolute minimum and often unstable  
  - 8 GB provides reliable startup and agent enrollment
- **Agent VM** can run comfortably with **1–2 GB RAM**

### Network Requirements

- **Both VMs must be on the same network**  
  (e.g. VMware NAT / `VMnet8`)
- **Agent must be able to reach the manager IP**
- No DNS tricks are required; IP-based communication is sufficient (IP-only is fine).

### Firewall Configuration (Manager VM)

The manager must allow inbound traffic from agents.  
If Ubuntu UFW is enabled, the following ports are required:

```bash
# Agent enrollment (required)
sudo ufw allow 1515/tcp

# Agent communication
sudo ufw allow 1514/tcp
sudo ufw allow 1514/udp

# Optional (dashboard & indexer access)
sudo ufw allow 5601/tcp
sudo ufw allow 9200/tcp

sudo ufw reload
```

Without these rules, agents will fail to enroll even if services are running.

## Installing the Agent (Client VM)

### 1. Download the Wazuh agent package

Run this on the **agent VM**:

```bash
wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.14.1-1_amd64.deb
```

### 2. Install and register the agent

Replace `YOUR_MANAGER_IP` with the actual IP address of your Wazuh manager.

```bash
sudo WAZUH_MANAGER="YOUR_MANAGER_IP" \
     WAZUH_AGENT_NAME="ubuntu-client-01" \
     dpkg -i wazuh-agent_4.14.1-1_amd64.deb
```

### 3. Enable and start the agent service

```bash
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```

Verify:

```bash
sudo systemctl status wazuh-agent
```

## Agent Enrollment (Required Step)

Installing the agent is **not** enough.  
Each agent must authenticate with the manager.

### 4. Enroll the agent with the manager

Run on the **agent VM**:

```bash
sudo /var/ossec/bin/agent-auth -m YOUR_MANAGER_IP
```

Successful output:

```text
INFO: Authorization successful.
```

Then restart the agent:

```bash
sudo systemctl restart wazuh-agent
```

## Verification

### On the Manager VM

Check registered agents:

```bash
sudo /var/ossec/bin/agent_control -lc
```

Expected output:

```text
ID: 001, Name: ubuntu-client-01, Active
```

### Dashboard

- Open the Wazuh Dashboard  
- Navigate to **Agents**  
- You should now see **1 active agent**

## Common Pitfalls (Lessons Learned)

- **Agent service running ≠ agent registered**
- **`wazuh-authd` may be running but UFW can silently block TCP/1515**
- **Manager startup on low RAM can delay agent enrollment**

Always verify:

- **Manager is running**
- **Port 1515 is listening**
- **Firewall allows inbound agent traffic**

