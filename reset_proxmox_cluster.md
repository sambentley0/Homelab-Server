# 💥 Reset and Rebuild a Proxmox VE Cluster from Scratch

This guide walks through the **complete reset** of a Proxmox cluster and cleanly re-adding nodes to a fresh cluster configuration. Useful when dealing with broken quorum, invalid `corosync.conf`, or failed joins.

---

## 🔁 Overview

- One node (e.g. `pve-green-machine`) will be the **new cluster leader**
- Other nodes (e.g. `pve2-lenovo-machine`) will be **reset** and **re-added**
- Intel NUC (`abfh-nfs-host`) is used as a **quorum-only** node — not running containers or VMs, but participating in quorum

---

## 🧼 Step 1: Wipe Cluster State from Proxmox Nodes

> ⚠️ Do this only on **Proxmox nodes** — **not** on the NUC.

### 1. Stop Cluster Services

```bash
systemctl stop pve-cluster
systemctl stop corosync
```

### 2. Delete Old Cluster Configs

```bash
rm -rf /etc/pve/corosync.conf
rm -rf /etc/corosync/*
rm -rf /var/lib/corosync/*
rm -rf /var/lib/pve-cluster/*
```

### 3. Start Temporary Local Cluster Filesystem

```bash
pmxcfs -l
```

> Leave this running or background it with `&`

---

## 🔧 Step 2: Create the New Cluster on the Leader Node

> Run on `pve-green-machine`:

```bash
pvecm create abfhcluster
```

You might need to kill the exisitng local version:

```bash
killall -9 pmxcfs
```


Then restart cluster services:

```bash
systemctl restart corosync
systemctl restart pve-cluster
```

Check quorum:

```bash
pvecm status
```

You should see:

- `Quorate: Yes`
- `Expected votes: 1`

---

## 🔗 Step 3: Rejoin Other Proxmox Nodes

> On `pve2-lenovo-machine` or others:

1. **Reboot** the node (important for clean state):

```bash
reboot
```

2. **Join the cluster**:

```bash
pvecm add <leader-node-ip>
```

Example:

```bash
pvecm add 192.168.xxx.xxx
```

Enter the **root password of the leader node** when prompted.

---

## ✅ Step 4: Confirm Cluster Status

On either node:

```bash
pvecm status
```

You should see both nodes listed, quorum OK, and each reachable in the Proxmox web UI.

---

## 🧹 Step 5: Add Intel NUC as Quorum-Only Node

> This node runs **Ubuntu Server**, not Proxmox.

### 1. Copy `corosync.conf` and `authkey` from the leader

On the NUC, move them to a **temporary folder** like `/tmp/cluster/`, then move them with root permissions:

```bash
sudo mv /tmp/cluster/corosync.conf /etc/corosync/
sudo mv /tmp/cluster/authkey /etc/corosync/
sudo chmod 400 /etc/corosync/authkey
sudo chown root:root /etc/corosync/authkey
```

### 2. Check Hostname and `/etc/hosts`

Make sure hostname and local IP match:

```bash
hostnamectl set-hostname abfh-nfs-host
```

Then edit `/etc/hosts`:

```plaintext
127.0.0.1       localhost
192.168.xxx.xxx     abfh-nfs-host
```

Each node in the cluster **must be defined** in every node's `/etc/hosts`.

### 3. Enable and Start Corosync

```bash
systemctl enable corosync
systemctl start corosync
```

Check status:

```bash
systemctl status corosync
```

---

## 💪 Troubleshooting Tips

### ❌ "No valid name found for local host"

- Caused by hostname mismatch or missing entry in `/etc/hosts`
- Fix by setting correct hostname and IP in `/etc/hosts`

### ❌ "parse error in config: Missing closing brace"

- Your `/etc/corosync/corosync.conf` is malformed
- Fix: Check formatting, braces, and indentation. Make sure all nodes are listed properly.

### ❌ Can't modify files due to `pmxcfs` lock

- Ensure `pmxcfs -l` is running to allow local file access
- Or fully stop and recreate the cluster:

```bash
systemctl stop pve-cluster
systemctl stop corosync
pmxcfs -l
```

### ⚠️ NUC not showing up in Proxmox config?

- That’s normal — the NUC is **not a Proxmox node**, so it doesn’t appear in the web UI
- It should appear in `corosync.conf` and `pvecm status` output

---

## 🗳️ Optional: Give the NUC a Vote

By default, quorum-only nodes **do not have a vote**.

To give the NUC a vote (use cautiously — only do this if needed for quorum during failures):

```bash
corosync-cfgtool -R
corosync-cmapctl -s nodelist.node.X.quorum_votes -v 1
```

Replace `X` with the correct index of the NUC in the `corosync.conf` node list (starts at 0).

---

You're now fully reset and rebuilt. Let me know if you'd like to add Home Assistant, containers, or shared storage setup next.

