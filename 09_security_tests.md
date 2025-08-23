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
