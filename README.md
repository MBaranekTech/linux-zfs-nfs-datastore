# 🗄️ Ubuntu 24.04 LTS – Software RAID, ZFS & NFS Datastore Setup

> A step-by-step guide to installing Ubuntu Server 24.04 with software RAID (mdadm), and configuring ZFS and NFS for a reliable network-accessible storage solution.

This project documents a **real-world deployment** where Ubuntu Server was installed with software RAID 1 during installation, and **ZFS + NFS** were configured post-installation to create a simple, resilient NFS datastore.

---

## 🧰 Technologies Used

- **Ubuntu 24.04 LTS Server**
- **mdadm** (for software RAID)
- **ZFS** (with `zfsutils-linux`)
- **NFS** (via `nfs-kernel-server`)

---

## 🚀 Use Case

This setup is ideal for:
- Home or work **lab environments**
- Creating a **shared ZFS-backed datastore** over NFS
- Hosting storage for **VMs**, containers, or backups your infrastructure 
- Demonstrating your **Linux system administration** skills

---

## 🔧 Step 1 – Install Ubuntu 24.04 with Software RAID

> 💡 RAID is created **during installation** using Ubuntu's installer.

### 1. Boot Ubuntu Server 24.04 Installer

- Select "Try or Install Ubuntu Server"
- Choose your language, keyboard layout, and configure your network.

---

### 2. Configure Storage with Custom Layout

1. **Choose "Custom storage layout"**
2. Select **two identical disks** (e.g., `/dev/sda`, `/dev/sdb`)
3. If you have more disks and you want create RAID 10 for example you need indentify specific disks - every disk has own UUID use command ```sudo blkid```
4. **Crucial step** On **each disk**, create:
   - **EFI/BOOT partition** 
   - **Remaining space** as **Leave unformated**

5. Select **Create software RAID (mdadm)**:
   - RAID Level: `1` (Mirror)
   - Devices: RAID partitions from both disks
   - Name: `md0`

6. After creating your RAID device (e.g., `/dev/md0`), select it to create volumes.
7. Create Volume Group

- Name the volume group (e.g., `vg0`)
- Format Logical Volumes

- For `lv-root` → Use `ext4` -> /
- For `lv-swap` → Use `swap area` -> SWAP

---

### 3. Finish Installation

- Create user, configure SSH (optional), install updates
- Reboot into the installed system
- Install updates
---

## 🧪 Verify RAID

```bash
cat /proc/mdstat
sudo mdadm --detail /dev/md0
lsblk
```

🌊 Step 2 – Install ZFS and Prepare Remaining Disks
1. Install ZFS Tools
```
sudo apt update
sudo apt install zfsutils-linux -y
```
2. Identify Unused Disks
```
lsblk
```
3. Create ZFS Pool
```
sudo zpool create storage mirror /dev/sdc /dev/sdd
```

Replace "storage" with your preferred pool name.

4. Verify ZFS Pool
```
zpool status
zfs list
```
📁 Step 3 – Create ZFS Dataset for NFS
```
sudo zfs create storage/data
sudo zfs set mountpoint=/storage/data storage/data
```

📡 Step 4 – Set Up NFS Server
1. Install NFS Server
```
sudo apt install nfs-kernel-server -y
```
2. Export ZFS Dataset
```
Edit /etc/exports:
sudo nano /etc/exports
Add:
/storage/data 192.168.1.0/24(rw,sync,no_subtree_check)
```
Adjust the subnet/IP as needed for your LAN.

3. Apply Export Configuration
```
sudo exportfs -a
sudo systemctl restart nfs-server
```
🧪 Step 5 – Test NFS Share from a Client

On a Linux client - second server:
```
sudo apt install nfs-common -y
sudo mount <server_ip>:/tank/data /mnt
df -h /mnt

Set Up Permanent Mount on Client (using `/etc/fstab`)
To ensure the NFS share mounts automatically on client reboot, add an entry to the client’s `/etc/fstab`:
sudo nano /etc/fstab
Add this line:
<server_ip>:/storage/data   /mnt   nfs   defaults   0   0
```
```
RAID Health: cat /proc/mdstat or sudo mdadm --detail /dev/md0
ZFS Health: zpool status, zfs list
NFS Logs: journalctl -u nfs-server
```
🙋 About This Project

This configuration was built and tested in a real work environment.
I created this repository to showcase my system administration skills and document a reproducible storage setup using open-source tools.
