# Add extra 250 GB storage in root (/) or /dev/mapper/ubuntu--vg-ubuntu--lv

1. Identify the New Disk: Confirm the identifier of the new 250 GB storage disk by running:

        fdisk -l
2. Create a Physical Volume (PV) on the New Disk: in my case /dev/sdb 250 GB is identified

        pvcreate /dev/sdb
3. Extend the Volume Group (VG): Add the new physical volume to the existing volume group. Assuming your volume group is named ubuntu-vg, run:

        vgextend ubuntu-vg /dev/sdb
4. Extend the Logical Volume (LV): Extend the logical volume to include the new storage.
   
       lvextend -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv
5. Resize the Filesystem: Resize the filesystem to use the additional space:

        resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv
5. Verify: Confirm that the additional storage has been added:

        df -h
