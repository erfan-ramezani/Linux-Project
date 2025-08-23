# 12 – Automated Port Scan & Secure Report Site

## Overview
This task builds a secure system for **regularly scanning servers** with `nmap` and publishing the reports on a **protected HTTPS website**.  
Admins can log in via **Basic Auth** and view reports by date/time.  

---

## Requirements
- Debian 12 / Ubuntu 24.04  
- Tools: `nmap`, `cron`, `nginx`, `htpasswd` (apache2-utils)  
- TLS certificate from **Let's Encrypt**  
- Only authorized users can access reports  
- Each report must be timestamped and kept for history  

---

## Steps

### 1. Install dependencies
```bash
sudo apt update
sudo apt install -y nmap nginx apache2-utils certbot python3-certbot-nginx

### 2. Create scan script
sudo mkdir -p /var/www/portscan/reports
sudo tee /usr/local/bin/portscan.sh > /dev/null <<'EOF'
#!/bin/bash
# Port scan script with timestamped reports

TARGETS="127.0.0.1 example.com 192.168.1.10"
DATE=$(date +%F_%H-%M)
OUTPUT="/var/www/portscan/reports/scan_$DATE.html"

echo "<html><head><title>Port Scan Report - $DATE</title></head><body>" > "$OUTPUT"
echo "<h1>Port Scan Report ($DATE)</h1><pre>" >> "$OUTPUT"

# Run nmap and append results
nmap -sS -Pn $TARGETS >> "$OUTPUT"

echo "</pre></body></html>" >> "$OUTPUT"
EOF

sudo chmod +x /usr/local/bin/portscan.sh
### 3. Automate with cron
echo "0 */6 * * * root /usr/local/bin/portscan.sh" | sudo tee /etc/cron.d/portscan

### 4. Secure the site with Basic Auth
sudo htpasswd -c /etc/nginx/.htpasswd admin
# (enter password)
### 5. Configure Nginx site
sudo tee /etc/nginx/sites-available/portscan >/dev/null <<'EOF'
server {
    listen 80;
    server_name reports.example.com;

    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    server_name reports.example.com;

    root /var/www/portscan;
    index index.html;

    ssl_certificate /etc/letsencrypt/live/reports.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/reports.example.com/privkey.pem;

    location / {
        auth_basic "Restricted Access";
        auth_basic_user_file /etc/nginx/.htpasswd;
        autoindex on;   # Allow listing of reports
    }
}
EOF

sudo ln -s /etc/nginx/sites-available/portscan /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx
### 6. Obtain TLS certificate
sudo certbot --nginx -d reports.example.com
### Verification
Run manually:
sudo /usr/local/bin/portscan.sh
→ Reports appear in /var/www/portscan/reports/scan_<DATE>.html
Visit in browser:
https://reports.example.com/reports/
→ Login with Basic Auth, see all timestamped reports.
