# Unified Utility Container Setup (DuckDNS + OpenDNS/ddclient) on Proxmox

This guide will walk you through creating a unified LXC container on Proxmox that will run background services for DuckDNS and OpenDNS (via ddclient), providing dynamic DNS updates and basic filtering functionality for your home lab.

---

## 🛠️ Step 1: Create the LXC Container in Proxmox

1. Download the Ubuntu 22.04 LXC template (if not already done):
   - Proxmox Web UI → **Node** → **Local storage** → **CT Templates**
   - Click **Templates** → Find `ubuntu-22.04-standard` → Download

2. Create the container:
   - **Datacenter → Create CT**
     - **Node**: Choose your node (e.g., pve-node1)
     - **CT ID**: 200 (or next available)
     - **Hostname**: `utils-client`
     - **Password**: Set a secure root password
     - **Template**: `ubuntu-22.04-standard`
     - **Storage**: `nfs-nuc` (or `local-lvm` if preferred)
     - **Disk Size**: 4 GB minimum
     - **CPU**: 1 core
     - **Memory**: 512–1024 MB
     - **Network**:
       - Bridge: `vmbr0`
       - IP: Static or DHCP (set static if preferred)
   - Confirm and create the container

3. Start the container and open the console.

---

## ⚙️ Step 2: Install Base Packages

In the console or via SSH:

```bash
apt update && apt upgrade -y
apt install curl cron nano wget -y
```
