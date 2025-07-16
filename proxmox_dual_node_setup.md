# Proxmox Node Setup Guide (with NFS Storage and Quorum-Only NUC)

This guide walks you through setting up your two main Proxmox nodes, which will run all VMs and containers. Shared storage is provided by the Intel NUC over NFS, and the NUC also serves as a quorum-only node for improved cluster reliability.

---

## 🛠️ Part 1: What You’ll Need

- 2 desktop PCs with:
  - Intel CPUs (64-bit)
  - At least 8 GB RAM (16+ GB recommended)
  - SSD or HDD for OS installation
- Proxmox VE ISO: [https://www.proxmox.com/en/downloads](https://www.proxmox.com/en/downloads)
- USB stick (4GB+) to flash installer
- Access to Proxmox Web UI via browser

---

## 💽 Part 2: Install Proxmox VE

1. Flash the Proxmox ISO using Rufus or balenaEtcher
2. Boot each desktop and press `F12`/`F10` to choose the USB
3. Install Proxmox VE:
   - Choose ext4 or ZFS (ext4 is fine for OS-only use)
   - Set hostname (e.g., `pve-node1`, `pve-node2`)
   - Set static IP addresses
   - Create login password and email
4. After install, access the Web UI at:
   ```
   https://<node-ip>:8006
   ```

---

## 🔗 Part 3: Create Cluster

1. On **pve-node1**:
```bash
pvecm create mycluster
```

2. On **pve-node2**:
```bash
pvecm add <IP-of-node1>
```

3. On the **NUC (Ubuntu Server)**:
   - Install Corosync:
   ```bash
   sudo apt install corosync
   ```
   - Join the cluster:
   ```bash
   pvecm add <IP-of-node1>
   ```
   - Verify:
   ```bash
   pvecm status
   ```

---

## 📡 Part 4: Mount NFS Storage from the NUC

On both Proxmox nodes:

1. Go to: **Datacenter → Storage → Add → NFS**
2. Fill in:
   - **ID**: `nfs-nuc`
   - **Server**: `<IP of NUC>`
   - **Export**: `/nfs`
   - **Content**: Disk image, ISO, VZDump backup file
   - **Nodes**: All
3. Save and verify the storage appears under `Datacenter → Storage`

> 🧠 The NUC hosts this NFS share using a ZFS RAID 1 pool

---

## 🖥️ Part 5: VM and Container Usage

- All VM disks and backups should use the `nfs-nuc` storage
- Avoid using local storage (`local-lvm` or `local`) for VM disks
- Create or migrate VMs using:
  - Proxmox Web UI → Create VM → Hard Disk → Storage: `nfs-nuc`

---

## ♻️ Part 6: High Availability (Optional)

1. Enable HA services:
   - Go to **Datacenter → HA**
   - Create HA groups with both nodes
2. Assign critical VMs to HA groups
3. Proxmox will manage automatic failover if one node goes offline

> The NUC ensures quorum remains valid so failover can occur

---

## 🧼 Part 7: NUC Role Restrictions

In Proxmox Web UI:

- Go to **Datacenter → Storage**
- For each NFS entry, uncheck the NUC node
- Confirm the NUC is not assigned any:
  - Disk image
  - ISO
  - Backup
  - Container storage

✅ This ensures it is **quorum-only** and not used for compute or storage

---

## ✅ Final Architecture Overview

| Node        | OS                  | Role                          |
|-------------|---------------------|-------------------------------|
| pve-node1   | Proxmox VE          | Runs VMs & containers         |
| pve-node2   | Proxmox VE          | Runs VMs & containers         |
| nuc-nfs     | Ubuntu Server 22.04 | ZFS RAID 1 NFS + quorum-only ✅ |

---

This architecture ensures full **high availability**, **centralized shared storage**, and a **resilient Proxmox cluster** using three low-cost, flexible machines.
