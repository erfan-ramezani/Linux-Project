# 04 – Nginx + WordPress + TLS + Subdomain

## Requirements
- Domain: example.com, files.example.com
- DNS pointing to server
- Nginx + PHP-FPM + MariaDB installed

## Steps
1. Install Nginx, PHP-FPM, MariaDB  
2. Create WordPress DB + user  
3. Deploy WordPress to `/var/www/example.com`  
4. Configure Nginx vhosts with HTTPS, gzip, security headers  
5. Create subdomain `files.example.com` for file serving  
6. Enable TLS with Certbot  

## Commands
```bash
# Install web stack
sudo apt install -y nginx php-fpm mariadb-server certbot python3-certbot-nginx

# Create DB
sudo mysql -e "CREATE DATABASE wpdb;"
sudo mysql -e "CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'password';"
sudo mysql -e "GRANT ALL PRIVILEGES ON wpdb.* TO 'wpuser'@'localhost'; FLUSH PRIVILEGES;"

# WordPress setup
sudo mkdir -p /var/www/example.com
cd /var/www/example.com
sudo wget https://wordpress.org/latest.tar.gz
sudo tar -xzf latest.tar.gz --strip-components=1

# Certbot TLS
sudo certbot --nginx -d example.com -d www.example.com -d files.example.com

# Reload nginx
sudo nginx -t && sudo systemctl reload nginx
