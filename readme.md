<img width="1068" height="553" alt="image" src="https://github.com/user-attachments/assets/5f6f954b-0b5c-460f-828f-60417017cbd0" />

# Step 1 — Extend the LVM Partition
## Install growpart
    apt install cloud-guest-utils -y
## Now extend partition 3
    growpart /dev/sda 3
# Step 2 — Resize Physical Volume
## Now expand the LVM physical volume:
    pvresize /dev/sda3
## Check free space:
    pvs
# Step 3 — Extend Logical Volume
## Extend root LV to use all free space:
    lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv
# Step 4 — Resize Filesystem
## If using ext4 (default Ubuntu):
    resize2fs /dev/ubuntu-vg/ubuntu-lv
# Final Check
    df -h
<img width="1081" height="561" alt="image" src="https://github.com/user-attachments/assets/6f73c7d5-9ff4-4962-b733-aa91002be743" />

