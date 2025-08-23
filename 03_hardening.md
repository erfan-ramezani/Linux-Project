#  03 – Hardening with Lynis
پ
## Requirements
- Install **Lynis** for auditing.  
- Reach a security score of **> 75**.  
- Apply basic security improvements:  
  - Fail2ban (SSH brute-force protection)  
  - Auditd (audit logs)  
  - AIDE (integrity checker)  
  - AppArmor (mandatory access control)  
  - SSH hardening  
  - Kernel parameter hardening  

## Steps
1. **Install Lynis**  
   - Available via `apt`.  

2. **Run baseline audit**  
   - Save output for review.  

3. **Apply improvements**  
   - Harden kernel with sysctl.  
   - Install Fail2ban, Auditd, AIDE.  
   - Configure SSH to disable root/password login.  

4. **Re-run Lynis**  
   - Repeat until score > 75.  

## Commands
```bash
# Install Lynis
sudo apt install -y lynis

# Run baseline audit
sudo lynis audit system | tee ~/lynis_initial.txt

# Install common hardening tools
sudo apt install -y fail2ban auditd aide apparmor-utils

# Enable services
sudo systemctl enable --now fail2ban auditd

# Kernel hardening config
cat <<'EOF' | sudo tee /etc/sysctl.d/99-hardening.conf
net.ipv4.conf.all.rp_filter=1
net.ipv4.tcp_syncookies=1
net.ipv4.icmp_echo_ignore_broadcasts=1
net.ipv4.conf.all.accept_redirects=0
net.ipv6.conf.all.accept_redirects=0
kernel.randomize_va_space=2
kernel.kptr_restrict=2
kernel.unprivileged_bpf_disabled=1
EOF

# Apply kernel changes
sudo sysctl --system

# SSH hardening
sudo sed -i 's/^#*PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config
sudo sed -i 's/^#*PasswordAuthentication.*/PasswordAuthentication no/' /etc/ssh/sshd_config
sudo systemctl restart ssh

# Re-run Lynis
sudo lynis audit system | tee ~/lynis_after.txt
