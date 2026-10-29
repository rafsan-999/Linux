# Extend Ubuntu Root (`/`) LVM Volume with an Additional Disk

A step-by-step guide to adding a new disk (e.g. **250 GB**) to the root filesystem on Ubuntu systems that use **LVM** (Logical Volume Manager), specifically the default logical volume:

```
/dev/mapper/ubuntu--vg-ubuntu--lv
```

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Prerequisites](#-prerequisites)
- [Procedure](#-procedure)
  - [Step 1: Identify the New Disk](#step-1-identify-the-new-disk)
  - [Step 2: Create a Physical Volume](#step-2-create-a-physical-volume-pv)
  - [Step 3: Extend the Volume Group](#step-3-extend-the-volume-group-vg)
  - [Step 4: Extend the Logical Volume](#step-4-extend-the-logical-volume-lv)
  - [Step 5: Resize the Filesystem](#step-5-resize-the-filesystem)
  - [Step 6: Verify](#step-6-verify)
- [Quick Reference](#-quick-reference)
- [Troubleshooting](#-troubleshooting)

---

## 🔎 Overview

| Component            | Name / Value                          |
|----------------------|---------------------------------------|
| New disk             | `/dev/sdb` (250 GB)                   |
| Volume Group (VG)    | `ubuntu-vg`                           |
| Logical Volume (LV)  | `ubuntu-lv`                           |
| LV device path       | `/dev/mapper/ubuntu--vg-ubuntu--lv`   |
| Mount point          | `/`                                   |
| Filesystem           | `ext4`                                |

> **Note:** Device names vary between systems. Always confirm your own disk, VG, and LV names before running any command.

---

## ✅ Prerequisites

- Root or `sudo` privileges
- A new, **unused** disk attached to the server (physical or virtual)
- `lvm2` installed (included by default on Ubuntu Server)
- A recent **backup** or snapshot of the system

> ⚠️ **Warning:** `pvcreate` will erase any existing data on the target disk. Double-check the device name before proceeding.

---

## 🛠 Procedure

### Step 1: Identify the New Disk

List all disks and confirm the identifier of the new 250 GB disk:

```bash
sudo fdisk -l
```

Alternatively, use `lsblk` for a cleaner view:

```bash
lsblk
```

The new disk should appear with **no partitions** and **no mount point**. In this guide, it is identified as `/dev/sdb`.

---

### Step 2: Create a Physical Volume (PV)

Initialize the new disk as an LVM physical volume:

```bash
sudo pvcreate /dev/sdb
```

Confirm the physical volume was created:

```bash
sudo pvs
```

---

### Step 3: Extend the Volume Group (VG)

Add the new physical volume to the existing volume group (`ubuntu-vg`):

```bash
sudo vgextend ubuntu-vg /dev/sdb
```

Verify the free space now available in the volume group:

```bash
sudo vgs
```

> **Tip:** If your volume group has a different name, find it with `sudo vgdisplay`.

---

### Step 4: Extend the Logical Volume (LV)

Allocate all free space in the volume group to the root logical volume:

```bash
sudo lvextend -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv
```

> **Tip:** To add a specific amount instead of all free space, use `-L`, for example `-L +100G`.

---

### Step 5: Resize the Filesystem

Grow the filesystem to use the newly allocated space. This can be done online, without unmounting `/`.

**For `ext4` (Ubuntu default):**

```bash
sudo resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv
```

**For `xfs`:**

```bash
sudo xfs_growfs /
```

> **Tip:** Check your filesystem type with `df -Th /`.

---

### Step 6: Verify

Confirm the root filesystem now reflects the additional storage:

```bash
df -h /
```

You can also review the full LVM layout:

```bash
sudo lsblk
sudo lvs
```

---

## ⚡ Quick Reference

```bash
# 1. Identify the new disk
sudo fdisk -l

# 2. Create a physical volume
sudo pvcreate /dev/sdb

# 3. Extend the volume group
sudo vgextend ubuntu-vg /dev/sdb

# 4. Extend the logical volume
sudo lvextend -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv

# 5. Resize the filesystem (ext4)
sudo resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv

# 6. Verify
df -h /
```

> **Shortcut:** Steps 4 and 5 can be combined with the `-r` (`--resizefs`) flag:
>
> ```bash
> sudo lvextend -r -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv
> ```

---

## 🧰 Troubleshooting

| Issue | Possible Cause | Solution |
|-------|----------------|----------|
| New disk not visible in `fdisk -l` | Disk not detected by the kernel | Rescan the SCSI bus: `echo "- - -" \| sudo tee /sys/class/scsi_host/host*/scan` |
| `pvcreate` fails: *device excluded by a filter* | Disk contains an old partition table or signature | Wipe it with `sudo wipefs -a /dev/sdb` (⚠️ destroys data) |
| `vgextend` fails: *volume group not found* | Incorrect VG name | Run `sudo vgs` to find the correct name |
| `resize2fs` fails: *bad magic number* | Filesystem is not `ext4` | Check with `df -Th /` and use `xfs_growfs` for XFS |
| `df -h` shows no change | Filesystem was not resized | Re-run Step 5 |

---

## 📄 License

This guide is provided as-is under the [MIT License](LICENSE). Use at your own risk and always back up your data before modifying storage.
