# 05 – Firewall & SSH Hardening (iptables)

## Requirements
- استفاده از **iptables** برای کنترل ورودی و خروجی.
- فقط پورت‌های 22، 80 و 443 باز باشند.
- ورود SSH فقط با کلید عمومی (key-only)، روت غیرفعال.
-آیپی Whitelist برای SSH (مثلاً فقط به IP خاص اجازه بدی).

## Steps & Commands

```bash
# Reset rules
sudo iptables -F
sudo iptables -X
sudo iptables -t nat -F
sudo iptables -t nat -X
sudo iptables -t mangle -F
sudo iptables -t mangle -X

# Set default policies
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT DROP

# Allow localhost traffic
sudo iptables -A INPUT -i lo -j ACCEPT
sudo iptables -A OUTPUT -o lo -j ACCEPT

# Allow established connections
sudo iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A OUTPUT -m state --state RELATED,ESTABLISHED -j ACCEPT

# Allow web (HTTP, HTTPS)
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT
sudo iptables -A OUTPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A OUTPUT -p tcp --dport 443 -j ACCEPT

# SSH from whitelist only (example: 203.0.113.0/24)
sudo iptables -A INPUT -p tcp --dport 22 -s 203.0.113.0/24 -j ACCEPT
sudo iptables -A OUTPUT -p tcp --sport 22 -d 203.0.113.0/24 -j ACCEPT

# Drop everything else
sudo iptables -A INPUT -j DROP
sudo iptables -A OUTPUT -j DROP

# Make persistent (Ubuntu/Debian)
sudo apt install -y iptables-persistent
sudo netfilter-persistent save
sudo netfilter-persistent reload

# SSH hardening (outside iptables)
sudo sed -i 's/^#*PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config
sudo sed -i 's/^#*PasswordAuthentication.*/PasswordAuthentication no/' /etc/ssh/sshd_config
sudo systemctl restart ssh
