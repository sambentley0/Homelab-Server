# Proxmox Home Server Cluster with NUC-Backed NFS and Quorum Node

This setup creates a robust, low-cost home lab infrastructure using two Proxmox nodes for compute and a third Intel NUC serving as both an NFS-based shared storage server (via ZFS RAID 1) and a quorum-only node to maintain high availability.

---

## 🧱 System Architecture

| Device       | Role                        | OS                      | Description |
|--------------|-----------------------------|--------------------------|-------------|
| **Node 1**   | Proxmox compute node         | Proxmox VE               | Runs VMs and containers |
| **Node 2**   | Proxmox compute node         | Proxmox VE               | Runs VMs and containers |
| **Intel NUC**| NFS server + quorum-only     | Ubuntu Server 22.04 LTS  | Hosts shared storage and ensures quorum |

---

## 🗂️ Storage Design

- The **Intel NUC** hosts a **ZFS RAID 1 pool** using two USB-connected 2.5" SATA SSDs
- The pool is mounted at `/nfs` and exported over the network via **NFS**
- Both Proxmox nodes mount the `/nfs` export and store:
  - VM disk images
  - ISO images
  - Backup files
- NUC boots from a separate M.2 SATA SSD (not part of the RAID)

---

## 🔁 High Availability

- The NUC ensures **quorum is maintained** when one compute node goes offline
- With HA groups configured in Proxmox, VMs can be automatically migrated or restarted on the remaining node

---

## 🔌 Networking

- All devices are on the same **LAN subnet**
- Static IPs are assigned to all nodes for reliable cluster communication
- NFS traffic runs over the same network as Proxmox Corosync communication

---

## 🛠️ Key Services to Be Deployed

These will run on the two Proxmox nodes and be stored on the shared NFS:

- 🏠 **Home Assistant** (VM)
- 🔧 **Node-RED** (Container or VM)
- 📡 **Mosquitto MQTT** (Container)
- 💾 Optional backup, media, or monitoring tools

---

## ✅ Key Benefits

- Low power consumption (NUC and USB-based SSDs)
- Centralized, mirrored storage with **ZFS**
- Simple quorum enforcement with 3-node cluster
- Easily expandable VM and service capacity
- Full Proxmox feature set including **snapshots, backups, and HA**

---

## 📦 Guide Summary

This system is designed to be:
- **Resilient** – survives node failure via quorum and mirrored storage
- **Modular** – VMs and containers can be migrated freely
- **Low-maintenance** – with automated backups and monitoring
- **Efficient** – with low idle power and smart resource division

---

## 📁 Related Setup Guides

- `nuc_nfs_setup.md` – NUC setup for ZFS + NFS + quorum
- `proxmox_nodes_setup.md` – Setup for both Proxmox compute nodes
- `home_assistant_proxmox_setup.md` – Deploying Home Assistant as VM