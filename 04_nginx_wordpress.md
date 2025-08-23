## Requirements
- Domain names:  
  - `example.com` (main WordPress site)  
  - `files.example.com` (file server)  
- DNS pointing to server IP.  
- Services: Nginx, PHP-FPM, MariaDB, Certbot.  

## Steps
1. **Install services**  
   - Nginx, PHP-FPM, MariaDB, Certbot.  

2. **Setup database**  
   - Create `wpdb` and a user with privileges.  

3. **Deploy WordPress**  
   - Extract files to `/var/www/example.com`.  

4. **Configure Nginx vhost**  
   - Redirect HTTP → HTTPS.  
   - Enable gzip + security headers.  

5. **Create file server subdomain**  
   - `/var/www/files.example.com` with optional autoindex.  

6. **TLS certificates**  
   - Use Certbot to issue and install certificates.  

## Commands
```bash
# Install services
sudo apt install -y nginx php-fpm mariadb-server certbot python3-certbot-nginx

# Database setup
sudo mysql -e "CREATE DATABASE wpdb;"
sudo mysql -e "CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'StrongPassword123!';"
sudo mysql -e "GRANT ALL PRIVILEGES ON wpdb.* TO 'wpuser'@'localhost'; FLUSH PRIVILEGES;"

# WordPress deployment
sudo mkdir -p /var/www/example.com
cd /var/www/example.com
sudo wget https://wordpress.org/latest.tar.gz
sudo tar -xzf latest.tar.gz --strip-components=1
sudo chown -R www-data:www-data /var/www/example.com

# Nginx config for WordPress
sudo tee /etc/nginx/sites-available/example.com >/dev/null <<'EOF'
server {
  listen 80;
  server_name example.com www.example.com;
  return 301 https://$host$request_uri;
}

server {
  listen 443 ssl http2;
  server_name example.com www.example.com;

  root /var/www/example.com;
  index index.php index.html;

  include snippets/fastcgi-php.conf;
  location ~ \.php$ {
    fastcgi_pass unix:/run/php/php-fpm.sock;
  }

  ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
}
EOF

# File server subdomain
sudo mkdir -p /var/www/files.example.com
sudo chown -R www-data:www-data /var/www/files.example.com
sudo tee /etc/nginx/sites-available/files.example.com >/dev/null <<'EOF'
server {
  listen 80;
  server_name files.example.com;
  return 301 https://$host$request_uri;
}

server {
  listen 443 ssl http2;
  server_name files.example.com;

  root /var/www/files.example.com;
  autoindex on;

  ssl_certificate /etc/letsencrypt/live/files.example.com/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/files.example.com/privkey.pem;
}
EOF

# Enable sites
sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/files.example.com /etc/nginx/sites-enabled/

# Obtain certificates
sudo certbot --nginx -d example.com -d www.example.com -d files.example.com

# Reload nginx
sudo nginx -t && sudo systemctl reload nginx
