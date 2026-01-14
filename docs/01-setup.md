
# Wazuh Setup (Lab)

This guide installs the Wazuh manager, indexer, and dashboard on a single Ubuntu VM using the official quickstart installer.
This setup is intended for learning and lab purposes, not production.

## Environment
- OS: Ubuntu (64-bit)
- Deployment: Single host (manager, indexer, dashboard on one VM)
- Hypervisor: VMware / VirtualBox / Hyper-V

## VM Requirements
- 2+ vCPU
- 4+ GB RAM (8 GB recommended if possible)
- 25+ GB disk
- Internet access

## Prerequisites
- Fresh Ubuntu VM
- sudo access
- curl installed
- No other package installations running (apt, Software Updater)

⚠️ On fresh Ubuntu installs, background updates may temporarily lock APT.  
If the installer fails, reboot and wait a few minutes before retrying.

## Install Wazuh (Single Host)
1) SSH into the Ubuntu VM.  
2) Download and run the installer:
```bash
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh
sudo bash ./wazuh-install.sh -a
```

**Command explanation**

- **`curl -sO`**: Downloads the `wazuh-install.sh` script from Wazuh without verbose output (`-s`) and saves it using the original filename (`-O`).
- **`sudo bash ./wazuh-install.sh -a`**: Runs the installer with **all-in-one mode**:
  - `-a`: Installs **manager, indexer, and dashboard** on the same VM (ideal for lab use).
3) When the installer finishes, note:
- Dashboard URL
- Generated admin password

4) Access the Wazuh dashboard:
   - Find your VM's IP address:
   ```bash
   ip a
   ```
   Look for the IP address under your network interface (usually `eth0` or `ens33`).

   **Command explanation**

   - **`ip a`**: Shows all network interfaces and their IP addresses.  
     - In most labs, you’ll use the IPv4 address on your main interface (often in a `192.168.x.x` or `10.x.x.x` range).
   
   - Open a web browser and navigate to:
   ```
   https://<VM-IP>
   ```
   Replace `<VM-IP>` with the IP address you found.
   
   - Log in with:
   - **Username**: `admin`
   - **Password**: The password shown at installer completion
   
   - Accept the self-signed certificate warning (this is expected in a lab environment).
   
   You should now see the Wazuh dashboard interface.

![Wazuh dashboard overview](../images/01_images/wazuh_dashboard_start.png)

## Validation
Confirm all services are running:
```bash
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard
```

**Command explanation**

- **`systemctl status <service>`**: Shows whether a service is **active (running)**, stopped, or failed, plus recent log lines.
- **`wazuh-manager`**: Core manager component (rules, analysis, agent handling).
- **`wazuh-indexer`**: Stores and indexes events (OpenSearch backend).
- **`wazuh-dashboard`**: Web UI that queries the indexer and manager.
Expected result: `Active: active (running)`  
Successful login to the web dashboard confirms a valid installation.

### Service status examples

![Wazuh manager status](../images/01_images/status_wazuh_manager.png)

![Wazuh indexer status](../images/01_images/status_wazuh_indexer.png)

![Wazuh dashboard status](../images/01_images/status_wazuh_dashboard.png)

## Change default dashboard password
After confirming you can log in with the generated `admin` password, change it to something you control:
```bash
sudo /usr/share/wazuh-indexer/plugins/opensearch-security/tools/wazuh-passwords-tool.sh \
  -u admin -p '<your_password>'
sudo systemctl restart wazuh-dashboard
```
Then log back in to the dashboard using the new password.

**Command explanation**

- **`wazuh-passwords-tool.sh`**: Utility to change credentials for Wazuh components.
- **`-u admin`**: Specifies the **user account** whose password you are changing.
- **`-p '<your_password>'`**: The **new password** to set (replace with your own secure value).
- **`systemctl restart wazuh-dashboard`**: Restarts the web UI so it picks up the new credentials.

## Troubleshooting (Installer Issues)
**Check for APT / dpkg lock before running the installer**  
```bash
ps aux | grep -E "apt|dpkg"
```
If you see processes like `unattended-upgrades` or `apt.systemd.daily`, wait for them to finish or reboot the VM.

**Command explanation**

- **`ps aux`**: Lists all running processes with details.
- **`grep -E "apt|dpkg"`**: Filters for package-management processes (APT, dpkg).  
  - If they appear, the package manager is busy and the installer may fail with a lock error.

**Installer fails due to APT lock**  
If the installer reports `Another process is using APT`:
- `sudo reboot`
- After reboot, wait 2–3 minutes, then retry the installer.

**Clean up a failed or partial installation**  
If the installer fails mid-way or components are left in a broken state:
```bash
sudo systemctl stop wazuh-indexer wazuh-manager wazuh-dashboard 2>/dev/null
sudo apt remove --purge -y wazuh-* filebeat opensearch-dashboard opensearch
sudo rm -rf /var/lib/wazuh /var/ossec /etc/wazuh /usr/share/wazuh
sudo rm -rf /etc/opensearch /var/lib/opensearch
sudo apt autoremove -y
sudo apt --fix-broken install -y
```
Notes: package “not found” errors are expected if components never fully installed.

**Command explanation**

- **`systemctl stop ...`**: Stops any Wazuh-related services that might still be running.
- **`apt remove --purge -y wazuh-* ...`**: Uninstalls Wazuh, Filebeat, and OpenSearch packages and their configuration (`--purge`), auto-accepting prompts (`-y`).
- **`rm -rf /var/lib/... /etc/...`**: Deletes leftover data/config directories to fully clean the install.
- **`apt autoremove -y`**: Cleans up unused dependencies.
- **`apt --fix-broken install -y`**: Asks APT to repair any broken package dependencies.

**“Wazuh indexer already installed” error**  
If the installer reports `ERROR: Wazuh indexer already installed`, force a clean overwrite:
```bash
sudo bash ./wazuh-install.sh -a -o
```
This removes existing Wazuh components and reinstalls everything.

**Command explanation**

- **`-a`**: All-in-one deployment (manager, indexer, dashboard).
- **`-o`**: **Overwrite mode** – removes any existing Wazuh installation and performs a fresh install.  
  - Useful when a previous install is corrupted or partially removed.

**Verify service status after install/reinstall**  
```bash
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard
```
Expected: `Active: active (running)`. Successful dashboard login confirms a good install.

**Check installer logs**  
If installation fails again, inspect:
```bash
sudo less /var/log/wazuh-install.log
```

## Optional: VMware Guest Tools
If running on VMware:
```bash
sudo apt update
sudo apt install -y open-vm-tools open-vm-tools-desktop
sudo reboot
```

## Notes
- Ubuntu ISO: [ubuntu.com/download/desktop](https://ubuntu.com/download/desktop)
- Installer reference: [Wazuh Quickstart](https://documentation.wazuh.com/current/quickstart.html)
- Credentials and certificates are stored in the installer output bundle (`wazuh-install-files.tar`)
- This lab uses a single-node deployment for simplicity
- Production environments typically separate components

