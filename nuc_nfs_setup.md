# Intel NUC NFS Server + Proxmox Quorum-Only Node Setup (Ubuntu Server 22.04 LTS)

This guide walks you through setting up your Intel NUC6i3SYH as a **dedicated NFS server** with **ZFS RAID 1** and adds it as a **quorum-only node** to your Proxmox cluster. This ensures shared storage for VM/container workloads and improves cluster stability without running VMs on the NUC itself.

---

## 🛠️ Part 1: What You’ll Need

- Intel NUC6i3SYH
- 1x M.2 SATA SSD (boot drive)
- 2x identical 2.5" SATA SSDs (ZFS mirror via USB 3.0 enclosures)
- USB stick (4GB+) with Ubuntu Server 22.04 LTS
- Ethernet cable (for static LAN connection)
- Access to another computer for flashing the installer

---

## 💽 Part 2: Install Ubuntu Server 22.04

1. Download the ISO: https://ubuntu.com/download/server
2. Flash it to USB using balenaEtcher or Rufus
3. Boot NUC and press `F10` to select USB stick
4. Install Ubuntu Server:
   - Choose minimal installation
   - Install to the **M.2 SATA SSD** (do not install ZFS root)
   - Set a **static IP** (e.g., `192.168.1.13`)
   - Set hostname (e.g., `nuc-nfs`)
   - Choose strong credentials

---

## 🔧 Part 3: Install ZFS and Create RAID 1 Pool

1. Install ZFS:
```bash
sudo apt update
sudo apt install zfsutils-linux
```

2. Identify your external SSDs:
```bash
lsblk
```

3. Create a ZFS mirror (RAID 1) using the USB-attached SSDs:
```bash
sudo zpool create -m /nfs nfsdata mirror /dev/sda /dev/sdb
```

*(Replace with actual device paths.)*

4. Confirm:
```bash
zpool status
```

---

## 📡 Part 4: Set Up the NFS Server

1. Install NFS tools:
```bash
sudo apt install nfs-kernel-server
```

2. Edit exports:
```bash
sudo nano /etc/exports
```

Add:
```
/nfs 192.168.1.0/24(rw,sync,no_subtree_check,no_root_squash)
```

3. Apply:
```bash
sudo exportfs -a
sudo systemctl restart nfs-kernel-server
```

---

## 🔗 Part 5: Join the Proxmox Cluster (Quorum-Only Node)

1. On Proxmox Node 1:
```bash
pvecm add <NUC-IP>
```

2. On the NUC:
```bash
sudo apt install corosync
pvecm add 192.168.1.11
```

3. Confirm:
```bash
pvecm status
```

4. In the Proxmox Web UI:
   - Don’t assign the NUC any storage or VM roles
   - Leave it as **quorum-only**

---

## 🔁 Part 6: Add NFS Storage to Proxmox

On both Proxmox nodes:

1. Go to: **Datacenter → Storage → Add → NFS**
2. Configure:
   - ID: `nfs-nuc`
   - Server: `<NUC IP>`
   - Export: `/nfs`
   - Content: Disk image, ISO, VZDump backup file
   - Nodes: All
3. Save and verify availability

---

## 🧪 Part 7: Test & Monitor

- Confirm `/nfs` appears in Proxmox storage list
- Test creating a VM or backup
- Run:
```bash
zpool status
zpool scrub nfsdata
```

---

## ✅ System Summary

| Component | Role |
|----------|------|
| OS       | Ubuntu Server 22.04 LTS |
| Storage  | 2x SSDs in ZFS RAID 1 via USB |
| NFS Path | `/nfs` |
| Cluster  | Quorum-only Proxmox node (no VMs run here) |

---

## 🔄 What if SSD #1 (boot) fails?

If the **boot SSD** fails:

1. Reinstall Ubuntu Server on a new SSD (M.2 or internal)
2. Reinstall ZFS:
```bash
sudo apt install zfsutils-linux
```
3. Re-import your pool:
```bash
sudo zpool import nfsdata
```

✅ Your data remains safe  
✅ NFS share is preserved  
✅ No pool rebuild needed

---

This setup provides **reliable shared storage and cluster stability** using the NUC — without running any VMs or containers on it.
