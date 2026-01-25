# Wazuh Setup (Lab)

This is how I set up Wazuh on a single Ubuntu VM. I'm using the official quickstart installer which puts the manager, indexer, and dashboard all on one machine - perfect for a lab environment.

## My Setup
- OS: Ubuntu (64-bit)
- Everything on one VM (manager, indexer, dashboard)
- Running on VMware (but VirtualBox or Hyper-V should work fine too)

## What You'll Need
- 2+ vCPU cores
- At least 4 GB RAM (8 GB is better if you can swing it)
- 25+ GB disk space
- Internet connection

## Before You Start
- Fresh Ubuntu VM
- sudo access
- curl installed (usually comes with Ubuntu)
- Make sure nothing else is installing packages (no apt or Software Updater running)

⚠️ Heads up: On a fresh Ubuntu install, background updates might lock APT. If the installer fails, just reboot and wait a few minutes before trying again.

## Installing Wazuh

1. SSH into your Ubuntu VM

2. Download and run the installer:
```bash
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh
sudo bash ./wazuh-install.sh -a
```

**What these commands do:**
- `curl -sO` downloads the installer script quietly and saves it with the original name
- `sudo bash ./wazuh-install.sh -a` runs it in all-in-one mode, which installs the manager, indexer, and dashboard on the same VM (great for labs)

3. When it finishes, the installer will show you:
- The dashboard URL
- An auto-generated admin password (write this down!)

4. Access the dashboard:
   - First, find your VM's IP address:
   ```bash
   ip a
   ```
   Look for the IP under your network interface (usually `eth0` or `ens33`).

   **What `ip a` does:**
   - Shows all your network interfaces and their IP addresses
   - In most labs, you'll want the IPv4 address on your main interface (usually something like `192.168.x.x` or `10.x.x.x`)

   - Open your web browser and go to:
   ```
   https://<VM-IP>
   ```
   Replace `<VM-IP>` with the IP you just found.

   - Log in with:
   - Username: `admin`
   - Password: The one the installer showed you

   - You'll get a self-signed certificate warning - that's normal for a lab, just accept it.

   You should now see the Wazuh dashboard!

![Wazuh dashboard overview](../images/01_images/wazuh_dashboard_start.png)

## Making Sure Everything Works

Check that all the services are running:
```bash
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard
```

**What `systemctl status` does:**
- Shows you if a service is running, stopped, or failed, plus recent log entries
- `wazuh-manager` is the core component (handles rules, analysis, and agents)
- `wazuh-indexer` stores and indexes events (uses OpenSearch)
- `wazuh-dashboard` is the web interface

You want to see `Active: active (running)` for all three. If you can log into the web dashboard, you're good to go!

### Service status examples

![Wazuh manager status](../images/01_images/status_wazuh_manager.png)

![Wazuh indexer status](../images/01_images/status_wazuh_indexer.png)

![Wazuh dashboard status](../images/01_images/status_wazuh_dashboard.png)

## Changing the Default Password

Once you've confirmed you can log in with the generated password, change it to something you'll remember:
```bash
sudo /usr/share/wazuh-indexer/plugins/opensearch-security/tools/wazuh-passwords-tool.sh \
  -u admin -p '<your_password>'
sudo systemctl restart wazuh-dashboard
```
Then log back in with your new password.

**What this does:**
- `wazuh-passwords-tool.sh` is the utility for changing Wazuh credentials
- `-u admin` tells it which user to change
- `-p '<your_password>'` is your new password (use your own!)
- `systemctl restart wazuh-dashboard` restarts the web UI so it picks up the new password

## Troubleshooting

**Check if APT is locked before running the installer**  
```bash
ps aux | grep -E "apt|dpkg"
```
If you see processes like `unattended-upgrades` or `apt.systemd.daily`, wait for them to finish or just reboot the VM.

**What this does:**
- `ps aux` lists all running processes
- `grep -E "apt|dpkg"` filters for package management processes
- If you see any, the package manager is busy and the installer might fail

**Installer fails with APT lock error**  
If you get `Another process is using APT`:
- Run `sudo reboot`
- After it comes back up, wait 2–3 minutes, then try the installer again

**Clean up a failed installation**  
If the installer failed partway through or things are broken:
```bash
sudo systemctl stop wazuh-indexer wazuh-manager wazuh-dashboard 2>/dev/null
sudo apt remove --purge -y wazuh-* filebeat opensearch-dashboard opensearch
sudo rm -rf /var/lib/wazuh /var/ossec /etc/wazuh /usr/share/wazuh
sudo rm -rf /etc/opensearch /var/lib/opensearch
sudo apt autoremove -y
sudo apt --fix-broken install -y
```
Don't worry if you see "package not found" errors - that just means those components never fully installed.

**What this cleanup does:**
- Stops any Wazuh services that might still be running
- Uninstalls all Wazuh, Filebeat, and OpenSearch packages and their configs
- Deletes leftover data and config directories
- Cleans up unused dependencies
- Fixes any broken package dependencies

**"Wazuh indexer already installed" error**  
If the installer says the indexer is already installed, force a clean reinstall:
```bash
sudo bash ./wazuh-install.sh -a -o
```
This removes everything and does a fresh install.

**What `-o` does:**
- `-a` is all-in-one mode (manager, indexer, dashboard)
- `-o` is overwrite mode - it removes any existing Wazuh installation and starts fresh
- Useful when a previous install got messed up or partially removed

**Check services after install/reinstall**  
```bash
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard
```
You want `Active: active (running)` for all three. If you can log into the dashboard, you're all set.

**Check installer logs if something goes wrong**  
If installation keeps failing, take a look at:
```bash
sudo less /var/log/wazuh-install.log
```

## Optional: VMware Guest Tools

If you're running this on VMware, you might want to install the guest tools for better integration:
```bash
sudo apt update
sudo apt install -y open-vm-tools open-vm-tools-desktop
sudo reboot
```

## Notes
- Ubuntu ISO: [ubuntu.com/download/desktop](https://ubuntu.com/download/desktop)
- Installer reference: [Wazuh Quickstart](https://documentation.wazuh.com/current/quickstart.html)
- The installer saves credentials and certificates in `wazuh-install-files.tar`
- This is a single-node setup for simplicity - production usually separates these components
