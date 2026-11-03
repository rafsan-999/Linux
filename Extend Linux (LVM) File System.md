# Extend a File System in Linux (LVM)

> All commands require root privileges (`sudo`). **Take a backup or snapshot before resizing disks.**

## 1. Check the current setup

### Disk space and filesystem type

```bash
df -Th
```

### Block devices (disks, partitions, LVM volumes)

```bash
lsblk
```

Include filesystem types:

```bash
lsblk -f
```

### Physical volumes (PV)

```bash
sudo pvs
sudo pvdisplay
```

### Volume groups (VG): check `VFree` for free space

```bash
sudo vgs
sudo vgdisplay
```

### Logical volumes (LV)

```bash
sudo lvs
sudo lvdisplay
```

> **LVM layout:** Disk → Physical Volume (PV) → Volume Group (VG) → Logical Volume (LV) → Filesystem

> `/dev/mapper/ubuntu--vg-ubuntu--lv` and `/dev/ubuntu-vg/ubuntu-lv` are two names for the same logical volume.

## 2. Extend using free space already in the volume group

If `sudo vgs` shows free space under `VFree`, extend the logical volume directly. This is common on Ubuntu, where the installer often leaves part of the disk unallocated.

### Add a specific amount

```bash
sudo lvextend -r -L +8G /dev/ubuntu-vg/ubuntu-lv
```

### Use all free space

```bash
sudo lvextend -r -l +100%FREE /dev/ubuntu-vg/ubuntu-lv
```

> `-r` (`--resizefs`) resizes the filesystem along with the volume and works for both ext4 and XFS. Without `-r`, you must resize the filesystem manually (see Section 6).

## 3. Extend by adding a new disk

### Step 1: Detect the new disk

If the disk was added to a running VM and doesn't appear in `lsblk`, rescan:

```bash
for host in /sys/class/scsi_host/host*; do echo "- - -" | sudo tee $host/scan; done
```

### Step 2: Identify the new disk

```bash
lsblk
```

Confirm the new disk (e.g. `/dev/sdb`) has the expected size and **no partitions or mount points**.

> ⚠️ Do **not** run `mkfs` on the disk. `pvcreate` prepares it for LVM.

### Step 3: Create a physical volume

```bash
sudo pvcreate /dev/sdb
```

### Step 4: Add it to the volume group

```bash
sudo vgextend ubuntu-vg /dev/sdb
```

### Step 5: Extend the logical volume and filesystem

```bash
sudo lvextend -r -l +100%FREE /dev/ubuntu-vg/ubuntu-lv
```

### Step 6: Verify

```bash
df -h
```

## 4. Extend after enlarging an existing disk

Use this when the existing disk was made bigger, for example a VM disk resized in VMware, Hyper-V, or a cloud console.

### Step 1: Rescan the disk so Linux sees the new size

```bash
echo 1 | sudo tee /sys/class/block/sda/device/rescan
```

### Step 2: Grow the partition (if the PV is on a partition)

Check with `lsblk`. If the PV is `/dev/sda3`, grow partition 3:

```bash
sudo growpart /dev/sda 3
```

> `growpart` is in the `cloud-guest-utils` package (Ubuntu/Debian) or `cloud-utils-growpart` (RHEL/CentOS).

### Step 3: Resize the physical volume

```bash
sudo pvresize /dev/sda3
```

### Step 4: Extend the logical volume and filesystem

```bash
sudo lvextend -r -l +100%FREE /dev/ubuntu-vg/ubuntu-lv
```

## 5. Extend without LVM

### Filesystem on a partition (e.g. `/dev/sdb1`)

Rescan the disk and grow the partition:

```bash
echo 1 | sudo tee /sys/class/block/sdb/device/rescan
sudo growpart /dev/sdb 1
```

Resize the filesystem (ext4):

```bash
sudo resize2fs /dev/sdb1
```

Or for XFS (use the mount point):

```bash
sudo xfs_growfs /mount/point
```

### Filesystem directly on the disk, with no partition (e.g. `/dev/sdb`)

```bash
echo 1 | sudo tee /sys/class/block/sdb/device/rescan
sudo resize2fs /dev/sdb
```

## 6. Resize the filesystem manually

Only needed if you ran `lvextend` **without** `-r`.

### ext4

```bash
sudo resize2fs /dev/ubuntu-vg/ubuntu-lv
```

### XFS (use the mount point)

```bash
sudo xfs_growfs /
```

## 7. Shrink a logical volume (use with caution)

> ⚠️ **Shrinking can destroy data.** Back up first.
>
> - **XFS cannot be shrunk.**
> - **ext4 must be unmounted** to shrink, so the root (`/`) volume can't be shrunk while the system is running from it.
> - Always use `-r` so the filesystem is shrunk **before** the volume. Running `lvreduce` without it corrupts the filesystem.

```bash
sudo umount /data
sudo lvreduce -r -L -8G /dev/data-vg/data-lv
sudo mount /data
```

## 8. Remove a disk from a volume group

### Remove a working disk

Move its data to other disks in the volume group (requires enough free space):

```bash
sudo pvmove /dev/sdb
```

Remove it from the volume group:

```bash
sudo vgreduce ubuntu-vg /dev/sdb
```

Remove the LVM label from the disk:

```bash
sudo pvremove /dev/sdb
```

### Remove a failed or missing disk

`--removemissing` removes physical volumes that are no longer present. Logical volumes stored on the missing disk will be lost.

```bash
sudo vgreduce --removemissing ubuntu-vg
```

## Quick Reference

| Task | Command |
|------|---------|
| Check disks and filesystems | `lsblk -f`, `df -Th` |
| Check free space in VG | `sudo vgs` |
| Create physical volume | `sudo pvcreate /dev/sdb` |
| Add disk to volume group | `sudo vgextend ubuntu-vg /dev/sdb` |
| Resize PV after disk enlarged | `sudo pvresize /dev/sda3` |
| Grow a partition | `sudo growpart /dev/sda 3` |
| Extend LV + filesystem by 8 GB | `sudo lvextend -r -L +8G /dev/ubuntu-vg/ubuntu-lv` |
| Extend LV + filesystem to all free space | `sudo lvextend -r -l +100%FREE /dev/ubuntu-vg/ubuntu-lv` |
| Resize ext4 filesystem | `sudo resize2fs /dev/ubuntu-vg/ubuntu-lv` |
| Resize XFS filesystem | `sudo xfs_growfs /mount/point` |
| Remove disk from VG | `sudo pvmove /dev/sdb` then `sudo vgreduce ubuntu-vg /dev/sdb` |