## 📂 docs/03_hardening.md
```markdown
# 03 – Hardening with Lynis (> 75)

## Requirements
- Lynis installed

## Steps
1. Install Lynis
2. Run baseline audit
3. Apply recommended fixes until score > 75

## Commands
```bash
sudo apt install -y lynis
sudo lynis audit system | tee ~/lynis_initial.txt
sudo apt install -y fail2ban auditd aide apparmor-utils
sudo systemctl enable --now fail2ban auditd

# Kernel hardening
cat <<'EOF' | sudo tee /etc/sysctl.d/99-hardening.conf
net.ipv4.conf.all.rp_filter=1
net.ipv4.tcp_syncookies=1
...
EOF
sudo sysctl --system

sudo lynis audit system | tee ~/lynis_after.txt
