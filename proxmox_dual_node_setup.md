# Proxmox VE Node Setup (for Dual-Node Cluster)

This guide walks you through installing and configuring Proxmox VE on **two separate PCs**, which will act as your main compute nodes for running virtual machines (VMs) and containers. These nodes will connect to a shared ZFS-backed NFS storage server (the NUC) previously configured.

---

## 🛠️ Part 1: Requirements

- 2 desktop PCs with:
  - Intel CPUs (compatible with virtualization)
  - Minimum 8 GB RAM (16+ GB recommended)
  - SSD or HDD for local Proxmox OS install
- Wired Ethernet connection on the same LAN
- Proxmox VE installer USB stick
- Static IP addresses for each node
- Access to the Intel NUC running NFS over `/nfs`

---

## 💽 Part 2: Install Proxmox VE on Each Node

Repeat the steps below on **each PC**:

1. Download the latest Proxmox ISO:  
   https://www.proxmox.com/en/downloads
2. Flash it to USB using Rufus or balenaEtcher.
3. Boot the PC, press the appropriate key to access the boot menu (usually F10/F12/ESC/DEL).
4. Select the USB stick and start the **Proxmox VE installer**.
5. Follow the prompts:
   - Accept license
   - Select your internal SSD/HDD for install
   - Choose `ext4` unless you want to test ZFS locally
   - Set hostname: `pve-node1` and `pve-node2`
   - Set static IPs: `192.168.1.11` and `192.168.1.12` (adjust to fit your LAN)
   - Set root password and email
6. Reboot and access the web UI via:  
   `https://<node-ip>:8006`

---

## 🔧 Part 3: Initial Node Configuration

1. Log in to each node's web UI using `root` and the password.
2. Update each node:
   - Web UI → **Datacenter → Node → Updates → Refresh → Upgrade**
   - Or via shell:
   ```bash
   apt update
   apt full-upgrade
   ```

---

## 🌐 Part 4: Add Shared NFS Storage

On **both nodes**:

1. Go to: **Datacenter → Storage → Add → NFS**
2. Set:
   - ID: `nfs-nuc`
   - Server: IP of your NUC
   - Export: `/nfs`
   - Content types: Disk image, ISO, Backup, Container template
   - Nodes: Select both nodes
3. Save and test storage visibility

---

## 🖧 Part 5: Cluster Creation

On **Node 1 (e.g., pve-node1)**:
```bash
pvecm create my-cluster
```

On **Node 2 (e.g., pve-node2)**:
```bash
pvecm add <IP-of-node1>
```
- This will synchronize configurations and enable centralized management.

After this, both nodes will appear in the **same web UI**, and you can manage them from either interface.

---

## 🧪 Part 6: Test Cluster and Storage

- Ensure both nodes appear under **Datacenter → Cluster**
- Create a test VM or container on one node
- Store it on the NFS volume
- Migrate it to the second node (right-click → Migrate)

---

## 📦 Summary

| Component | Role |
|----------|------|
| Node 1 | Proxmox VE | Host VMs/CTs |
| Node 2 | Proxmox VE | Host VMs/CTs |
| Shared Storage | NFS from NUC | Centralized VM data |
| Networking | LAN (Gigabit recommended) | Connects all nodes |

---

You're now ready to deploy Home Assistant, Node-RED, Mosquitto, and more across your Proxmox cluster using shared storage.

