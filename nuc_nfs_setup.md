# Intel NUC NFS Server Setup with ZFS RAID 1 (Ubuntu Server 22.04 LTS)

This guide will walk you through configuring your Intel NUC as a dedicated NFS server with mirrored SSDs using ZFS RAID 1, to serve two Proxmox nodes with shared storage.

---

## 🛠️ Part 1: What You’ll Need

- Intel NUC6i3SYH
- 2x matching SSDs (SATA or M.2)
- USB stick (4GB+) with Ubuntu Server 22.04 LTS
- Ethernet cable for wired LAN
- Access to another computer for flashing the installer

---

## 💽 Part 2: Install Ubuntu Server 22.04

1. Download the ISO: https://ubuntu.com/download/server
2. Flash it to USB using balenaEtcher or Rufus
3. Boot NUC and press `F10` to select the USB stick
4. Install Ubuntu Server:
   - Choose minimal installation
   - Use one SSD for OS install (or create a small partition manually)
   - Set a static IP (recommended for NFS)
   - Set hostname (e.g. `nuc-nfs.local`)
   - Set a strong username and password

---

## 🔧 Part 3: Install ZFS and Create RAID 1 Pool

1. Install ZFS:
```bash
sudo apt update
sudo apt install zfsutils-linux
```

2. Identify your SSDs:
```bash
lsblk
```

3. Create a ZFS mirror (RAID 1) using the two SSDs:
```bash
sudo zpool create -m /nfs nfsdata mirror /dev/sda /dev/sdb
```
*(Replace `/dev/sda` and `/dev/sdb` with the actual device names)*

4. Verify the pool:
```bash
zpool status
```

---

## 📡 Part 4: Set Up the NFS Server

1. Install NFS server:
```bash
sudo apt install nfs-kernel-server
```

2. Configure NFS export:
```bash
sudo nano /etc/exports
```

Add the following line:
```
/nfs 192.168.1.0/24(rw,sync,no_subtree_check,no_root_squash)
```
*(Adjust the subnet to match your LAN)*

3. Apply the export:
```bash
sudo exportfs -a
sudo systemctl restart nfs-kernel-server
```

---

## 🔁 Part 5: Connect from Proxmox Nodes

On each Proxmox node:

1. Go to: **Datacenter → Storage → Add → NFS**
2. Set:
   - ID: `nfs-nuc`
   - Server: `<NUC IP>`
   - Export: `/nfs`
   - Content: Disk image, ISO, VZDump backup file
   - Nodes: (all)
3. Save

---

## 🧪 Part 6: Test and Monitor

- Confirm the NFS mount appears in Proxmox storage
- Test creating a VM or backup on the NFS share
- Check ZFS health:
```bash
zpool status
zpool scrub nfsdata
```

---

## 📦 Summary

| Component | Role |
|----------|------|
| NUC OS | Ubuntu Server 22.04 LTS |
| Storage | ZFS RAID 1 using two SSDs |
| Export | `/nfs` via NFS to Proxmox nodes |
| Purpose | Shared VM storage and backups |

---

## 🔄 What if SSD #1 (boot) fails?

If the boot SSD (running Ubuntu) fails:

1. Your ZFS RAID 1 pool (`/nfs`) is **untouched** and **fully intact**.
2. Simply reinstall Ubuntu Server on a new SSD (ext4, like before).
3. After booting into the new install, re-import your ZFS pool:

```bash
sudo apt install zfsutils-linux
sudo zpool import nfsdata
```

✅ Your NFS data will reappear exactly as before  
✅ No data loss  
✅ No RAID rebuild necessary  
✅ Easy recovery process
