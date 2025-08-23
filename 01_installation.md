# 01 – Installation

## Requirements
- OS: Ubuntu 24.04 LTS (preferred) or Debian 12
- Hardware: ≥2 vCPU, ≥4GB RAM, ≥60GB disk
- Root access or sudo user

## Steps
1. Boot ISO and select **manual partitioning**  
2. Create partitions:  
   - `/boot` → 2G, ext4, non-LVM  
   - VG=vg0 with LVM PV on remaining disk:  
     - `lv_root` → 30G (mount `/`, ext4)  
     - `lv_var` → 30G (mount `/var`, ext4)  
3. Finish installation and log in  

## Commands
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Essential tools
sudo apt install -y vim curl wget git htop ufw lvm2 net-tools gnupg ca-certificates unzip tar bsdmainutils

# Enable unattended upgrades (Ubuntu)
sudo apt install -y unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades
