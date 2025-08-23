# 06 – Disks & Mounts (/var expansion, /backup LV)

## Requirements
- Expand `/var` if more space is required
- Create new logical volume `/backup`
- Ensure mounts persist after reboot

## Steps
1. **Check current disk usage**  
   Use `lsblk` and `df -h` to identify partitions and volumes.  
2. **Extend /var**  
   If `/var` is running out of space, extend `lv_var`.  
3. **Create /backup volume**  
   Add a new LV (e.g. `lv_backup`) under `vg0`, format it, and mount at `/backup`.  
4. **Make persistent**  
   Add entries to `/etc/fstab` for auto-mount on reboot.  

## Commands
```bash
# Check current disk and VG
lsblk -f
sudo vgs
sudo lvs

# Extend /var by 10G
sudo lvextend -L +10G /dev/vg0/lv_var
sudo resize2fs /dev/vg0/lv_var

# Create /backup LV (20G example)
sudo lvcreate -L 20G -n lv_backup vg0
sudo mkfs.ext4 /dev/vg0/lv_backup
sudo mkdir -p /backup
sudo mount /dev/vg0/lv_backup /backup

# Verify
df -h | grep backup

# Persist in /etc/fstab
echo "/dev/vg0/lv_backup /backup ext4 defaults 0 2" | sudo tee -a /etc/fstab
