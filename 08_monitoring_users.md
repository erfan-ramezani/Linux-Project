# 08 – Monitoring & Restricted User

## Requirements
- Create a restricted user (no root privileges)
- Monitor system resources and services
- Send alerts if resource usage is high or service fails

## Steps
1. **Create restricted user**  
   - No sudo access  
   - Home directory with limited permissions  
2. **Monitoring tools**  
   Install `htop`, `glances`, or configure `monit`.  
3. **Service monitoring**  
   Use `monit` to watch Nginx, MariaDB, SSH, etc.  
4. **Alerts**  
   Configure email notifications for failures.  

## Commands
```bash
# Create restricted user
sudo adduser restricteduser
sudo usermod -L restricteduser  # lock password login
sudo usermod -s /bin/false restricteduser  # disable shell login

# Install monitoring tools
sudo apt install -y htop glances monit

# Configure monit (example for nginx)
sudo tee /etc/monit/conf.d/nginx <<'EOF'
check process nginx with pidfile /run/nginx.pid
  start program = "/bin/systemctl start nginx"
  stop program  = "/bin/systemctl stop nginx"
  if failed port 80 protocol http then alert
EOF

# Reload monit
sudo systemctl enable --now monit
sudo monit reload
sudo monit status
