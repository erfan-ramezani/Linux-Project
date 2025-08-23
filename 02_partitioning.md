## 📂 docs/02_partitioning.md
```markdown
# 02 – Partitioning (Verification)

## Requirements
- LVM present: vg0 with lv_root (30G) and lv_var (30G)

## Steps & Commands
```bash
lsblk -f
sudo vgs
sudo lvs -a -o +devices
sudo blkid
mount | egrep "(/ |/var )"
