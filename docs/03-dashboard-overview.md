# Wazuh Dashboard – Overview

This is my overview of the **Overview** page in the Wazuh dashboard. The Overview page gives you a quick summary of how your agents are doing and what alerts have been popping up recently. It's a good way to check that everything is working like it should.

---

## Agents Summary

### What it is

The **Agents Summary** panel shows you the connection status of all your enrolled Wazuh agents.

Agents are grouped by status, like:
- Active
- Disconnected
- Pending
- Never connected

This panel tells you whether your endpoints are **visible and talking** to the Wazuh manager. It's just about connectivity - it doesn't tell you about threats, alerts, or system health beyond whether they're connected.

![Agents Summary Panel](../images/03_images/agent_summary/agent_summary.png)

![Agents Table View](../images/03_images/agent_summary/agent_preview.png)

---

### Agent Details 

When you click on an individual agent from the Agents Summary panel, Wazuh shows you a detailed view with the **current state, activity level, and security context** for that specific endpoint.

![Agents Details View](../images/03_images/agent_summary/agent_details_1.png)
![Agents Details View](../images/03_images/agent_summary/agent_details_2.png)

---

#### Agent Identity and Status

This section gives you the basic info about the agent and whether it's healthy.

It includes:
- **Agent ID and name** - How Wazuh identifies this agent
- **Status** - Whether the agent is currently active
- **IP address** - Where the endpoint is on the network
- **Agent version** - What version of the Wazuh agent is installed
- **Operating system** - OS and version the agent is reporting
- **Group** - Which agent group it's in (this determines what config gets applied)
- **Cluster node** - Which manager node is handling this agent
- **Registration date** - When the agent was enrolled
- **Last keep-alive** - When the agent last checked in

**Why this matters**  
This section confirms that the agent is:
- Properly enrolled
- Actively communicating
- Sending up-to-date data

Basically, it tells you whether you can trust the data you're seeing below.

---

#### System Inventory

The System Inventory section shows basic hardware and host info that the agent collected.

It usually includes:
- CPU model and number of cores
- Available memory
- Hostname
- Serial number (often missing for VMs)

**Why this matters**  
Gives you context about the endpoint without needing an external inventory system.

**In a lab**  
Don't be surprised if you see low resource values and missing serial numbers - that's normal for virtual machines. The fact that this data is here at all means **inventory collection is working**.

---

#### Events Count Evolution

This section shows you a graph of how many events the agent has been generating over time.

**Why this matters**
- Shows overall activity level
- Highlights sudden spikes or weird increases in event generation

**In a lab**  
A mostly flat or low-activity graph is totally normal in a quiet lab environment.

---

#### MITRE ATT&CK

The MITRE ATT&CK section shows you how alerts from this agent map to **MITRE ATT&CK tactics**.

**Why this matters**
- Categorizes detection rules into high-level adversary behavior classes
- Gives you context on what *types* of behaviors the rules are catching

**In a lab**  
Seeing tactics like Defense Evasion or Privilege Escalation in a lab is pretty common and usually nothing to worry about.

---

#### Compliance

The Compliance section maps alerts and checks to compliance frameworks like **PCI DSS**.

**Why this matters**
- Shows which compliance control categories are associated with what you're seeing
- Helps with compliance-oriented visibility and reporting

**In a lab**  
If you see charts with data, that's expected and doesn't mean you're non-compliant or at risk.

---

#### Vulnerability Detection

The **Vulnerability Detection** section shows you known vulnerabilities found on the agent based on installed packages.

It includes:
- Counts of detected vulnerabilities grouped by severity (Critical, High, Medium, Low)
- A list of the most frequently affected packages, if any

**Why this matters**
- Shows you known software vulnerabilities affecting the endpoint
- Confirms that vulnerability scanning is enabled and running

**Important note**  
Detected vulnerabilities mean **potential exposure**, not that something has been exploited. And just because you don't see any findings doesn't mean the system is secure.

**In a lab**  
Zero or low findings are common in fresh or lightly used lab systems - don't expect this to reflect real-world exposure levels.

---

#### Security Configuration Assessment (SCA)

The **SCA: Latest scans** section shows results from recent configuration assessments against security benchmarks.

It includes:
- The benchmark or policy used (like CIS Ubuntu Linux Benchmark)
- When the last scan completed
- Number of passed, failed, and not applicable checks
- An overall compliance score

**Why this matters**
- Evaluates system configuration against hardening guidelines
- Gives you a baseline view of security posture

**Important note**  
Low scores are super common on default installations. This doesn't mean something is misconfigured or compromised.

**In a lab**  
Failing checks are totally expected unless you've explicitly hardened the system.

---

#### File Integrity Monitoring (FIM)

The **FIM: Recent events** section shows recent file integrity events detected on the agent.

It includes:
- When the event happened
- File path that was affected
- What action was performed (like modified)
- Rule description and severity level

**Why this matters**
- Tracks changes to monitored files and directories
- Gives you visibility into filesystem activity

**In a lab**  
Seeing integrity events in a lab environment is normal and actually confirms that file integrity monitoring is working.

---

### Agent Configurations

When you are in the Agent Details page, to view Agent Configurations, click on the top left of that page. This shows which security and monitoring features are enabled and managed for this agent.

![Agents Configuration View](../images/03_images/agent_summary/agent_configuration_1.png)
![Agents Configuration View](../images/03_images/agent_summary/agent_configuration_2.png)

---

#### Configuration Status and Groups

Top of the page, display which group is the agent in (default) and synchronization status indicating whether the agent is using the latest configuration.

---

#### Main Configurations

Contain core agent behaviour and communication settings

---
#### Auditing and Policy Monitoring
---
#### System Threads and Incident Response
---
#### Log data analysis
---
#### Cloud security monitoring


## Last 24 Hours Alerts

### What it is

The **Last 24 Hours Alerts** panel shows you how many alerts Wazuh generated in the last 24 hours.

Alerts are usually broken down by:
- Time
- Severity level

This panel gives you a quick look at **recent alert activity** and helps you spot sudden increases or weird patterns in alert volume.

Keep in mind - these are **signals generated by detection rules**, not confirmed security incidents.

![Last 24 Hours Alerts Panel](../images/03_images/last_24_hours_alerts/last_24_hours.png)
