# 13 – Trivy Vulnerability Scanner (Auto HTML Reports + Email Alerts + HTTPS)

## Overview
Continuously scan your servers and services with **Trivy**, generate **timestamped HTML reports**, and **email an alert** whenever **CRITICAL** vulnerabilities are found.  
Reports are published on a **protected HTTPS site** (Basic Auth + Let's Encrypt), and historical reports remain available for review.

---

## Requirements
- Debian 12 / Ubuntu 24.04
- Tools:
  - `trivy` (vulnerability scanner)
  - `jq` (for JSON parsing)
  - `cron` (scheduling 4× daily)
  - `nginx` + `apache2-utils` (Basic Auth)
  - `certbot` + `python3-certbot-nginx` (TLS)
  - `msmtp-mta` + `mailutils` (send email via SMTP)
- SSH key-based access to remote servers (optional, if you will scan remotes via SSH)
- A DNS name for the report site, e.g. `trivy.example.com`

---

## Step 1 — Install Dependencies
```bash
sudo apt update
# Trivy (official repo)
sudo apt install -y curl gnupg lsb-release ca-certificates apt-transport-https
curl -fsSL https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo gpg --dearmor -o /usr/share/keyrings/trivy.gpg
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -cs) main" | \
  sudo tee /etc/apt/sources.list.d/trivy.list
sudo apt update
sudo apt install -y trivy jq

# Web stack for publishing reports
sudo apt install -y nginx apache2-utils certbot python3-certbot-nginx

# Email tools (lightweight SMTP client)
sudo apt install -y msmtp-mta mailutils

## Step 2 — Configure Outgoing Email (msmtp)
sudo tee /etc/msmtprc >/dev/null <<'EOF'
# /etc/msmtprc (system-wide)
defaults
auth           on
tls            on
tls_trust_file /etc/ssl/certs/ca-certificates.crt
logfile        /var/log/msmtp.log

account        default
host           smtp.example.com
port           587
from           alerts@example.com
user           alerts@example.com
password       <APP_PASSWORD_OR_SMTP_PASSWORD>
EOF

sudo chmod 600 /etc/msmtprc
sudo touch /var/log/msmtp.log && sudo chmod 640 /var/log/msmtp.log

## Step 3 — Prepare Report Site Structure
sudo mkdir -p /var/www/trivy/reports
sudo chown -R www-data:www-data /var/www/trivy
#Create Basic Auth user (prompted for password):
sudo htpasswd -c /etc/nginx/.htpasswd admin

## Step 4 — Nginx vhost (HTTPS + Basic Auth)
sudo tee /etc/nginx/sites-available/trivy >/dev/null <<'EOF'
server {
    listen 80;
    server_name trivy.example.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    server_name trivy.example.com;

    root /var/www/trivy;
    index index.html;

    ssl_certificate     /etc/letsencrypt/live/trivy.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/trivy.example.com/privkey.pem;

    # Protect entire site
    location / {
        auth_basic "Restricted";
        auth_basic_user_file /etc/nginx/.htpasswd;
        autoindex on;   # list reports
    }
}
EOF

sudo ln -s /etc/nginx/sites-available/trivy /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx

# Obtain TLS certificate (DNS must point to this server)
sudo certbot --nginx -d trivy.example.com

## Step 5 — Targets List (optional for remote hosts)
Define which hosts to scan. Use localhost if only scanning this server.
sudo mkdir -p /etc/trivy
sudo tee /etc/trivy/hosts.txt >/dev/null <<'EOF'
# One host per line. Use IP or FQDN. 'localhost' means scan this machine.
localhost
#192.168.10.20
#app.example.com
EOF
For remote hosts, ensure trivy is installed on them or accessible via your automation. Scans run via SSH and pull results back.

## Step 6 — Scanner Script (HTML + JSON + Email on CRITICAL)
This script:
Scans each host’s filesystem (trivy fs) and, if Docker exists, its local images (trivy image).
Saves JSON and HTML reports under /var/www/trivy/reports/.
Emails if CRITICAL vulnerabilities exist (subject includes host & timestamp).
Keeps historical reports (*_YYYY-MM-DD_HH-MM.*).
sudo tee /usr/local/bin/trivy_scan.sh >/dev/null <<'EOF'
#!/usr/bin/env bash
set -euo pipefail

# --- Config ---
HOSTS_FILE="/etc/trivy/hosts.txt"
REPORT_DIR="/var/www/trivy/reports"
ALERT_EMAIL="secops@example.com"

# Optional: SSH user (for remote)
SSH_USER="${SSH_USER:-root}"
SSH_OPTS="-o BatchMode=yes -o StrictHostKeyChecking=accept-new"

DATE="$(date +%F_%H-%M)"
mkdir -p "$REPORT_DIR"

have_cmd() { command -v "$1" >/dev/null 2>&1; }

scan_local_fs() {
  local host="localhost"
  local json="$REPORT_DIR/${host}_fs_${DATE}.json"
  local html="$REPORT_DIR/${host}_fs_${DATE}.html"

  # JSON (for parsing)
  trivy fs --quiet --format json --severity CRITICAL,HIGH,MEDIUM,LOW / > "$json" || true

  # HTML (wrap table output)
  {
    echo "<html><head><title>Trivy FS Report - $host - $DATE</title></head><body>"
    echo "<h1>Trivy Filesystem Report ($host) – $DATE</h1><pre>"
    trivy fs --quiet --format table --severity CRITICAL,HIGH,MEDIUM,LOW /
    echo "</pre></body></html>"
  } > "$html"

  echo "$json"
}

scan_local_images() {
  local host="localhost"
  local html="$REPORT_DIR/${host}_images_${DATE}.html"

  if have_cmd docker; then
    {
      echo "<html><head><title>Trivy Image Reports - $host - $DATE</title></head><body>"
      echo "<h1>Trivy Docker Image Reports ($host) – $DATE</h1><pre>"
      docker images --format '{{.Repository}}:{{.Tag}}' | grep -v '<none>' | while read -r image; do
        echo "=== $image ==="
        trivy image --quiet --format table --severity CRITICAL,HIGH "$image" || true
        echo
      done
      echo "</pre></body></html>"
    } > "$html"
  fi
}

scan_remote_fs() {
  local host="$1"
  local json="$REPORT_DIR/${host}_fs_${DATE}.json"
  local html="$REPORT_DIR/${host}_fs_${DATE}.html"

  # Pull JSON from remote (for parsing)
  ssh $SSH_OPTS "${SSH_USER}@${host}" "trivy fs --quiet --format json --severity CRITICAL,HIGH,MEDIUM,LOW /" > "$json" || true

  # Pull TABLE and wrap as HTML
  {
    echo "<html><head><title>Trivy FS Report - $host - $DATE</title></head><body>"
    echo "<h1>Trivy Filesystem Report ($host) – $DATE</h1><pre>"
    ssh $SSH_OPTS "${SSH_USER}@${host}" "trivy fs --quiet --format table --severity CRITICAL,HIGH,MEDIUM,LOW /" || true
    echo "</pre></body></html>"
  } > "$html"

  echo "$json"
}

email_if_critical() {
  local json="$1"
  local host="$2"
  local count=0

  if [ -s "$json" ] && have_cmd jq; then
    count="$(jq '[.Results[]?.Vulnerabilities[]? | select(.Severity=="CRITICAL")] | length' "$json" 2>/dev/null || echo 0)"
  fi

  if [ "${count}" -gt 0 ]; then
    local top
    top="$(jq -r '.Results[]?.Vulnerabilities[]? | select(.Severity=="CRITICAL") |
          "\(.PkgName)  \(.VulnerabilityID)  Installed:\(.InstalledVersion)  Fixed:\(.FixedVersion//"N/A")"' \
          "$json" 2>/dev/null | head -n 20 || true)"

    {
      echo "CRITICAL vulnerabilities detected on host: ${host}"
      echo "Time: $(date)"
      echo "Count: ${count}"
      echo
      echo "Top findings:"
      echo "${top:-"(details in attached report)"}"
      echo
      echo "Server: ${host}"
    } | mail -s "[CRITICAL] Trivy findings on ${host} — ${DATE}" "${ALERT_EMAIL}"
  fi
}

main() {
  if [ ! -f "$HOSTS_FILE" ]; then
    echo "hosts file not found: $HOSTS_FILE"
    exit 1
  fi

  while read -r host; do
    # skip blanks/comments
    [[ -z "${host// }" || "$host" =~ ^# ]] && continue

    if [ "$host" = "localhost" ] || [ "$host" = "127.0.0.1" ]; then
      j="$(scan_local_fs)"
      scan_local_images || true
      email_if_critical "$j" "localhost"
    else
      j="$(scan_remote_fs "$host")"
      email_if_critical "$j" "$host"
    fi
  done < "$HOSTS_FILE"
}

main "$@"
EOF
sudo chmod +x /usr/local/bin/trivy_scan.sh

## Step 7 — Schedule 4× Daily via Cron
echo "0 */6 * * * root /usr/local/bin/trivy_scan.sh" | sudo tee /etc/cron.d/trivy_scan
sudo systemctl restart cron

## Step 8 — Verification
sudo /usr/local/bin/trivy_scan.sh
You should see timestamped reports in:
/var/www/trivy/reports/
  ├─ localhost_fs_YYYY-MM-DD_HH-MM.json
  ├─ localhost_fs_YYYY-MM-DD_HH-MM.html
  └─ localhost_images_YYYY-MM-DD_HH-MM.html (if docker is present)
Open the site in your browser:
https://trivy.example.com/reports/
Login with Basic Auth user you created.
Confirm you can browse historical reports by timestamp.

## Notes & Recommendations
Scope: trivy fs / scans OS packages and files; if you use Docker, image scans are included (table HTML).
Remote servers: Ensure trivy is installed remotely; the script pulls JSON & TABLE over SSH.
Email: Alerts are sent only if CRITICAL findings are present.
Security: Keep /etc/msmtprc with chmod 600; use app passwords if available.
Rotation: If reports grow large, add a cleanup cron (e.g., delete files older than 30 days).
echo "15 0 * * * root find /var/www/trivy/reports -type f -mtime +30 -delete" | sudo tee /etc/cron.d/trivy_cleanup
