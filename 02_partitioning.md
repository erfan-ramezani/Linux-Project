# 02 – Partitioning (Verification & Expansion)

## Requirements
- Logical Volumes `lv_root` (mount `/`) and `lv_var` (mount `/var`) must exist under `vg0`.
- `/boot` partition must be separate and not on LVM.
- Ability to create or resize Logical Volumes.
- Sufficient disk space for expansion or new volumes.

## Steps
1. **Inspect current layout**  
   Use `lsblk` and `vgs`, `lvs` to understand current setup (partitions, VG, LV).

2. **Verify mounts and filesystems**  
   Confirm that `/`, `/var`, and `/boot` are mounted correctly and have ext4 filesystems.

3. **Resize or extend partitions (optional)**  
   If needed, extend `lv_root` or `lv_var`, or create new LVs (e.g., for `/backup`).

4. **Test changes before rebooting**  
   Temporarily mount new or extended volumes to check everything works (e.g., `/mnt/temp`).

5. **Permanently configure mounts**  
   Update `/etc/fstab` with correct UUIDs or device paths so mounts persist across reboots.

## Commands
```bash
# Inspect block device layout
lsblk -f

# Check VG and LVs
sudo vgs
sudo lvs -a -o +devices

# Check UUIDs and filesystem types
sudo blkid

# Check current mounts and usage
df -hT

# Example: Extend /var by 15G
sudo lvextend -L +15G /dev/vg0/lv_var
sudo resize2fs /dev/vg0/lv_var

# Example: Create new LV for /backup (20G)
sudo lvcreate -L 20G -n lv_backup vg0
sudo mkfs.ext4 /dev/vg0/lv_backup
sudo mkdir -p /backup
sudo mount /dev/vg0/lv_backup /backup

# Confirm new mount
df -hT | grep backup

# Add to /etc/fstab using UUID (recommended)
UUID=$(sudo blkid -s UUID -o value /dev/vg0/lv_backup)
echo "UUID=${UUID} /backup ext4 defaults 0 2" | sudo tee -a /etc/fstab

# Test fstab entries
sudo mount -a
