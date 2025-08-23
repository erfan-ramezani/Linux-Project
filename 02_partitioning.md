
---

## 📂 docs/02_partitioning.md
```markdown
# 02 – Partitioning (Verification)

## Requirements
- Ensure **LVM** is configured correctly with Volume Group `vg0`
- Verify Logical Volumes exist:  
  - `lv_root` → mounted at `/`  
  - `lv_var` → mounted at `/var`  
- Confirm `/boot` is a separate partition  

## Steps
1. **Check block devices**  
   Use `lsblk` to view device hierarchy and mount points.  

2. **Verify LVM setup**  
   - Confirm `vg0` exists with required LVs.  
   - Check which devices back the LVs.  

3. **Check filesystem types and UUIDs**  
   Use `blkid` to verify ext4 and UUIDs.  

4. **Ensure proper mounts**  
   - `/` and `/var` should be separate.  
   - `/boot` must not be under LVM.  

5. **Review fstab**  
   Ensure persistence across reboots.  

## Commands
```bash
# Check block device hierarchy
lsblk -f

# Verify Volume Group and LVs
sudo vgs
sudo lvs -a -o +devices

# Check filesystem UUIDs
sudo blkid

# Check current mounts
mount | egrep "(/ |/var |/boot)"

# Show disk usage
df -h

# Example /etc/fstab entries
cat /etc/fstab
