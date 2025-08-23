
---

## 📂 docs/07_backup_strategy.md
```markdown
# 07 – Backup Strategy (DB, WordPress files, Offsite)

## Requirements
- Backup database (MariaDB/MySQL)
- Backup WordPress files under `/var/www/example.com`
- Store backups locally in `/backup`
- Sync backups offsite (e.g. remote server with rsync + SSH)

## Steps
1. **Database backup**  
   Use `mysqldump` to export databases daily.  
2. **WordPress files backup**  
   Compress `/var/www/example.com` directory.  
3. **Local storage**  
   Store backups in `/backup` with timestamped filenames.  
4. **Offsite sync**  
   Use `rsync` or `scp` to copy to remote backup server.  
5. **Automation**  
   Add cron job for daily backups.  

## Commands
```bash
# Database backup
sudo mysqldump -u root -p wpdb > /backup/wpdb_$(date +%F).sql

# Files backup
sudo tar -czf /backup/wpfiles_$(date +%F).tar.gz /var/www/example.com

# Offsite sync
rsync -avz /backup/ user@backupserver:/remote/backup/

# Example cron job (daily at 2AM)
echo "0 2 * * * root mysqldump -u root -pPASSWORD wpdb > /backup/wpdb_\$(date +\%F).sql && tar -czf /backup/wpfiles_\$(date +\%F).tar.gz /var/www/example.com && rsync -avz /backup/ user@backupserver:/remote/backup/" | sudo tee /etc/cron.d/wpbackup
