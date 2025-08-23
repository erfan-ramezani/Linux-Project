## 📂 docs/02_partitioning.md
```markdown
# 02 – Partitioning (Verification)

## Requirements
- Ensure LVM volumes are present:
  - vg0 with lv_root (30G)
  - lv_var (30G)

## Steps
1. Verify block devices  
2. Confirm LVM configuration  
3. Check filesystem mounts  
4. Review `/etc/fstab`  

## Commands
```bash
# Inspect block devices
lsblk -f

# Check LVM
sudo vgs
sudo lvs -a -o +devices

# Filesystems
sudo blkid

# Ensure mounts
mount | egrep "(/ |/var )"
