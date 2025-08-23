# 07 – Backup Strategy (Database, WordPress files, Offsite)

## Requirements
- Backup WordPress database (`wpdb`)
- Backup WordPress files (`/var/www/example.com`)
- Store backups locally in `/backup`
- Sync backups to offsite server for disaster recovery
- Automate the process with `cron`

## Steps
1. **Database Backup**
   - Use `mysqldump` to export WordPress database.  
   - Store backup file in `/backup` with a timestamped filename.  

2. **Files Backup**
   - Compress WordPress directory (`/var/www/example.com`) into a `.tar.gz` archive.  
   - Store archive in `/backup`.  

3. **Offsite Backup**
   - Use `rsync` or `scp` to copy backups to a remote server.  
   - Requires SSH key authentication for automation.  

4. **Automation**
   - Create a cron job to run daily backups at 2 AM.  
   - Ensure backup rotation or cleanup to avoid disk full issues.  

## Commands
```bash
# Create local backup directory
sudo mkdir -p /backup

# Database backup
mysqldump -u root -pPASSWORD wpdb > /backup/wpdb_$(date +%F).sql

# Files backup
tar -czf /backup/wpfiles_$(date +%F).tar.gz /var/www/example.com

# Offsite sync (replace user@remote-server)
rsync -avz /backup/ user@backupserver:/remote/backup/

# Cron job (daily 2 AM)
echo "0 2
