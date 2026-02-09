# Endpoint Security - Overview

Check for endpoint's security baseline and guidelines and see if everything is up to standard. We can scan and check for IOCs, and alerts related to FIM. 

![Endpoint Security Panel](../images/04_images/endpoint_sec_overview.png)

## Configuration Assessment

sca just scan to verify if the agents have any weaknesses and reduce the attack surface

There will be 3 options on the top left when u are in this page:

---

### Dashboard

When u get into this page, u have an option to choose which agent will you scan. After choosing and scanning the agent using the CIS benchmark, it will return a dashboard like this:

![Dashboard](../images/04_images/configuration_assessment/sca_dashboard.png)

Apparently the default given Ubuntu when i downloaded from the official page failed over 50% of the check.

from my research, it looks like CIS benchmarks are hardening standards, so its basically asking “Is this system hardened for a production, security-sensitive environment?” So therfore technically the VM is not insecured or anything like that.

You can click into each ID to see the rationale and desc on why its failing and what to fix.

---

### Inventory

Inventory is basically just showing each scan as an item, and simple results like sucessful and failed check. 

![Inventory](../images/04_images/configuration_assessment/sca_inventory.png)

---

### Events

Events is sort of like inventory, but a little bit more detailed, and showing every scan and events in bar charts in time order.

![Events](../images/04_images/configuration_assessment/sca_events.png)

When clicking to the magnifier next to the timestamp, you can have more detailed version of that scan

![Document Details](../images/04_images/configuration_assessment/sca_events_document_detail.png)

In this document details, u have the scan's detailed info

when u click View Surrounding Documents, it will navigate to the discover page and show all the event logs which is is the last 24 hours alerts panel.

but besides from that, I would say these three sub-pages within sca serves very similar purposes and it depends on how you want to view and observe for info, like agent name and manager, policy, description, etc.

## Malware Detection

detect malware, but not using scan like antivirus, but combining signals from multiple modules and applying rules

![Malware Detection Dashboard](../images/04_images/malware_detection/malware_detection_dashboard.png)

How it works:

 - FIM watches files and reports changes
 - Rules + IOCs decide if a change looks suspicious or known-bad
 - Rootcheck looks for rootkit-style behavior
    - E.g.: known rootkit signatures, suspicious system behaviors, hidden files, processes, or ports
    - Good for catching stealthy techniques
    - Can be noisy or outdated if not tuned
 - AV logs (ClamAV / Defender) are collected if present
 - Optional enrichment (VirusTotal, YARA) adds context

## File Integrity Monitoring

