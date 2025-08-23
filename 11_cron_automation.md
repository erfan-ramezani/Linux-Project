# 11 – Cron & Automation

## Requirements
- Task 1: At **00:01 on the first day of every week**, generate a disk usage report (`lsblk`, `df -h`) and store it in a file.  
- Task 2: After **every reboot**, copy a secret file to a safe location or send it to another server.  
- System: Debian 12 / Ubuntu 24.04.  
- Tool: `cron` (cron jobs + special strings like `@reboot`).  

---

## Concepts
- Cron is a daemon that schedules jobs automatically.  
- Special strings:  
- `@reboot` → run once after reboot.  
- `@daily`, `@weekly`, `@monthly`, … for common schedules.  

---

## Steps
### Task 1 – Weekly Disk Report
1. Create a script `/usr/local/bin/disk_report.sh`.  
 - Collect output of `lsblk` and `df -h`.  
 - Save to `/var/log/disk_report_YYYY-MM-DD.txt`.  

2. Add cron job to run every Sunday at 00:01.  

---

### Task 2 – Secret File Copy After Reboot
1. Create a script `/usr/local/bin/secret_copy.sh`.  
 - Copy a sensitive file (`/etc/secret.key`) to `/backup/` OR send it via `scp` to remote host.  

2. Add `@reboot` cron job to trigger this script on every restart.  

---

## Commands
```bash
# Ensure cron is installed and enabled
sudo apt update
sudo apt install -y cron
sudo systemctl enable --now cron


# 1. Create disk report script
sudo tee /usr/local/bin/disk_report.sh > /dev/null <<'EOF'
#!/bin/bash
DATE=$(date +%F_%H-%M)
OUTPUT="/var/log/disk_report_$DATE.txt"

{
echo "=== Disk Report on $(date) ==="
echo
lsblk -f
echo
df -hT
} > "$OUTPUT"
EOF

# Make it executable
sudo chmod +x /usr/local/bin/disk_report.sh


# 2. Add cron job: 00:01 on the first day of the week (Sunday)
echo "1 0 * * 0 root /usr/local/bin/disk_report.sh" | sudo tee /etc/cron.d/disk_report


# 3. Create secret copy script
sudo tee /usr/local/bin/secret_copy.sh > /dev/null <<'EOF'
#!/bin/bash
SRC="/etc/secret.key"
DEST="/backup/secret_$(date +%F_%H-%M).key"

# Local copy
cp -v "$SRC" "$DEST"

# Optional: send to remote backup server
# scp "$SRC" user@remote-server:/remote/backup/
EOF

# Make it executable
sudo chmod +x /usr/local/bin/secret_copy.sh


# 4. Add cron job: run on every reboot
echo "@reboot root /usr/local/bin/secret_copy.sh" | sudo tee /etc/cron.d/secret_copy

###Verification

Check cron jobs:
sudo crontab -l
sudo ls /etc/cron.d/

Manually test scripts:
sudo /usr/local/bin/disk_report.sh
sudo /usr/local/bin/secret_copy.sh

Check logs:
Disk reports in /var/log/disk_report_*.txt.
Secret copies in /backup/.
