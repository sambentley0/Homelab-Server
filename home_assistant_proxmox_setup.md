# Home Assistant VM Setup on Proxmox

This guide walks you through deploying Home Assistant OS as a virtual machine on your Proxmox cluster using the shared NFS storage hosted on the Intel NUC.

---

## 📦 Part 1: Download the Home Assistant OS Image

1. Visit the [official Home Assistant OS VM download page](https://www.home-assistant.io/installation/alternative#virtual-machines)
2. Download the **KVM/Proxmox (.qcow2) image**
3. Unzip the file to extract the `.qcow2` disk image

---

## 🖥️ Part 2: Upload Image to Proxmox

1. Use SCP or the Proxmox Web UI to upload the `.qcow2` file to your **NFS storage** (`nfs-nuc`)
   - Upload it under: `Datacenter → Storage → nfs-nuc → Disk Images`

---

## ⚙️ Part 3: Create a New VM

1. In Proxmox Web UI, click **Create VM**
2. Choose:
   - Name: `home-assistant`
   - Node: (either Proxmox node)
   - Storage: Leave as default (we’ll override later)
3. Skip ISO image selection
4. Under **System**:
   - BIOS: OVMF (UEFI)
   - Machine: q35
   - Enable QEMU Agent (optional)
5. Under **Hard Disk**:
   - Bus/Device: `VirtIO`
   - Storage: `nfs-nuc`
   - Use the `qm importdisk` command (see below) to attach the `.qcow2` image
6. Under **CPU**:
   - Cores: 2 (minimum)
7. Under **Memory**:
   - 2048–4096 MB (recommended)
8. Under **Network**:
   - Model: `virtio` or `vmxnet3`
   - Bridge: `vmbr0`

---

## 🧱 Part 4: Import the Disk Image

After creating the VM (e.g., ID 100), SSH into the Proxmox node and run:

```bash
qm importdisk 100 /path/to/home-assistant.qcow2 nfs-nuc
```

Then go back to the Web UI:

- Go to the VM → Hardware → Add → Existing Disk
- Choose the imported disk and attach it as `virtio0`

---

## 🚀 Part 5: Boot and Configure

1. Start the VM
2. Open the console from Proxmox
3. Wait for Home Assistant OS to finish initial setup (~5–10 mins)
4. Navigate to `http://<vm-ip>:8123` in your browser
   - Use your DHCP server to find the IP or assign a static IP

---

## 🔄 Migrating from Home Assistant OS on Raspberry Pi

If you're currently running Home Assistant OS on a Raspberry Pi (e.g. Pi 3B+), you can easily migrate your configuration to the new VM:

1. On your Raspberry Pi:
   - Go to **Settings → System → Backup**
   - Create a **full backup** and download the `.tar` file to your computer

2. On the new Home Assistant VM:
   - Complete onboarding (create a temporary user)
   - Go to **Settings → System → Backup**
   - Upload your `.tar` backup file
   - Restore the backup (you may need to reboot)

✅ This restores:
- Your Home Assistant configuration
- Add-ons and integrations
- Dashboard and user settings

> Note: IP address or hostname changes may affect integrations; adjust as needed.

---

## ✅ Final Notes

- Home Assistant OS runs great with 2 vCPUs and 2–4 GB RAM
- Store VM on NFS-backed ZFS storage for reliability
- Snapshots and backups can be easily automated through Proxmox
