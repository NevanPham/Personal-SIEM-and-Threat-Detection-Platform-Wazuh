
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
3) When the installer finishes, note:
- Dashboard URL
- Generated admin password

4) Access the dashboard:
```
https://<VM-IP>
Username: admin
Password: shown at installer completion
```
Accept the self-signed certificate warning.

## Validation
Confirm all services are running:
```bash
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard
```
Expected result: `Active: active (running)`  
Successful login to the web dashboard confirms a valid installation.

## Troubleshooting (Common Lab Issues)
**APT lock error**  
If the installer reports another process using APT:
- Reboot the VM
- Wait for background updates to finish
- Re-run the installer

**“Wazuh indexer already installed”**  
If a previous install failed mid-way, the indexer may remain partially installed. Fix by running:
```bash
sudo bash ./wazuh-install.sh -a -o
```
This overwrites any existing Wazuh components and performs a clean reinstall.

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

