## 📂 docs/04_nginx_wordpress.md
```markdown
# 04 – Nginx + WordPress + TLS + Subdomain

## Requirements
- Domain: example.com, files.example.com
- DNS records pointing to server

## Steps
1. Install Nginx, PHP-FPM, MariaDB
2. Create DB & user
3. Deploy WordPress
4. Configure Nginx vhosts with HTTPS, security headers, gzip
5. Enable subdomain for file server
6. Issue Let's Encrypt certs with certbot

## Commands (snippets)
```bash
sudo apt install -y nginx php-fpm mariadb-server certbot python3-certbot-nginx
sudo mysql -e "CREATE DATABASE wpdb; CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'password'; GRANT ALL PRIVILEGES ON wpdb.* TO 'wpuser'@'localhost'; FLUSH PRIVILEGES;"
sudo wget https://wordpress.org/latest.tar.gz -P /var/www/
sudo tar -xzf /var/www/latest.tar.gz -C /var/www/
sudo systemctl enable --now nginx
sudo certbot --nginx -d example.com -d www.example.com -d files.example.com
