## Adding a Wazuh Agent (Endpoint)

how i set up + enrolled a wazuh agent in my lab. agent runs on a separate ubuntu vm and sends stuff back to the manager.

## How It Works

wazuh is basically manager + agents.

### Manager (Already Running)

manager does the heavy lifting:
- receives logs/events from agents
- runs rules / correlation
- stores data in indexer
- dashboard reads from it

### Agent (What We're Adding)

agent runs on endpoints:
- collects logs + security events
- does fim (file integrity monitoring)
- sends data to manager
- shows up in dashboard

important: install agent on a separate vm. installing it on the manager kinda defeats the point.

## What You'll Need

### Virtual Machines

- 1x wazuh manager vm (from `01-setup.md`)
- 1x agent vm (ubuntu server/desktop is fine)

### Hardware Notes

manager vm:
- min: 4gb ram (can be unstable)
- recommend: 8gb ram (way smoother for startup/enrollment)

agent vm:
- 1–2gb ram is fine

### Network

- both vms need to talk to each other
- same nat / host-only network works
- ips are fine, no dns needed

### Firewall Setup (Manager VM)

if ufw is enabled on manager, open these ports before enrolling agents:

```bash
# Agent enrollment
sudo ufw allow 1515/tcp

# Agent communication
sudo ufw allow 1514/tcp
sudo ufw allow 1514/udp

sudo ufw reload
```

if 1515/tcp is blocked, agent enrollment will fail even if everything else is up.

what the ports are for:
- 1515/tcp = agent enrollment (`agent-auth` talks here)
- 1514/tcp + 1514/udp = agent logs/telemetry -> manager
- `sudo ufw reload` applies rules

## What You'll See in the Dashboard

if no agents yet, endpoints page is basically empty and tells u to deploy one.

once enrolled + connected, it should show up as Active with os/ip/version.

## Deploying the Agent (Using the Dashboard)

dashboard has a guided flow that generates install commands for ur agent os.

### Step 1: Start Agent Deployment

- open dashboard
- go to endpoints
- click deploy new agent

![Deploy new agent button](../images/02_images/deploy_agent.png)

### Step 2: Pick Your Platform

pick os + architecture for agent vm.

for ubuntu labs:
- linux
- deb amd64

dashboard will customize commands based on selection.

![Select platform and package](../images/02_images/select_package.png)

### Step 3: Enter Agent Name and Server Address

in dashboard enter:
- agent name: something like `ubuntu-client-01`
- server address: manager ip (private ip is fine, fqdn not needed for lab)

this is what the agent uses to talk to the manager.

### Step 4: Install the Agent (On the Agent VM)

dashboard shows install commands. run on the agent vm:

download agent package:

```bash
wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.14.1-1_amd64.deb
```

install + set manager ip + agent name (replace `YOUR_MANAGER_IP`):

```bash
sudo WAZUH_MANAGER="YOUR_MANAGER_IP" \
     WAZUH_AGENT_NAME="ubuntu-client-01" \
     dpkg -i wazuh-agent_4.14.1-1_amd64.deb
```

quick note:
- `WAZUH_MANAGER` = manager ip/host
- `WAZUH_AGENT_NAME` = how it shows up in dashboard (name it whatever)
- `dpkg -i` installs the `.deb`

### Step 5: Start the Agent Service

enable + start agent:

```bash
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```

check:

```bash
sudo systemctl status wazuh-agent
```

![Start agent service commands](../images/02_images/start_agent.png)

## Agent Enrollment (You Have to Do This!)

installing package alone isn’t enough. agent has to enroll/auth with manager.

run on agent vm:

```bash
sudo /var/ossec/bin/agent-auth -m YOUR_MANAGER_IP
```

notes:
- `agent-auth` = enrollment client
- `-m` = manager ip (should match what u used above/in dashboard)

expected:

```text
INFO: Authorization successful.
```

restart agent:

```bash
sudo systemctl restart wazuh-agent
```

## Making Sure It Worked

### On the Manager VM

list registered agents:

```bash
sudo /var/ossec/bin/agent_control -lc
```

`-lc` lists agents + connection status.

should see something like:

```text
ID: 001, Name: ubuntu-client-01, Active
```

### In the Dashboard

- go to endpoints
- check status is Active

agent should show up with os/ip/version info.

![Active agent in dashboard](../images/02_images/agent_active.png)

## Common Issues

- agent service running ≠ agent enrolled
- ufw blocking 1515/tcp
- manager low ram = enrollment slow/flaky

if agent doesn’t show up:

- make sure manager services are running
- check port 1515 is listening / not blocked
- check agent logs:

```bash
sudo tail -f /var/ossec/logs/ossec.log
```
