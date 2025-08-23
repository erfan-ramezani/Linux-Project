# 09 – Security Tests (SSL Labs A, SecurityHeaders A, Port Scans)

## Requirements
- Ensure only required ports are accessible externally:
  - 22 (SSH)  
  - 80 (HTTP)  
  - 443 (HTTPS)  
- Block all other ports from the outside world.
- Run external tests (Qualys SSL Labs, securityheaders.com).
- Verify firewall with **iptables** and port scan tools (`nmap`, `ss`).

## Steps
1. **Check listening ports**  
   - Use `ss -tulnp` to list open ports and services.  
   - Use `nmap` from an external machine to simulate real attack surface.  

2. **Allow only required ports with iptables**  
   - Default DROP policy.  
   - Explicit ACCEPT for SSH, HTTP, HTTPS.  
   - Loopback traffic must always be allowed.  

3. **Test firewall**  
   - Run `nmap` from outside.  
   - Only ports 22, 80, 443 should appear open.  

4. **Run security tests**  
   - Use [SSL Labs](https://www.ssllabs.com/ssltest/) for HTTPS configuration.  
   - Use [securityheaders.com](https://securityheaders.com/) to verify headers.  

## Commands
```bash
# List all listening ports and services
sudo ss -tulnp

# Example: scan from local (basic check)
sudo nmap -sS -Pn localhost

# Flush existing iptables rules
sudo iptables -F
sudo iptables -X

# Set default DROP policies
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT ACCEPT

# Allow loopback
sudo iptables -A INPUT -i lo -j ACCEPT
sudo iptables -A OUTPUT -o lo -j ACCEPT

# Allow established/related connections
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# Allow SSH (22), HTTP (80), HTTPS (443)
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Save firewall rules (Debian/Ubuntu)
sudo apt install -y iptables-persistent
sudo netfilter-persistent save

# Verify rules
sudo iptables -L -v -n
