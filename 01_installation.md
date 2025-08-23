# 01 – Installation

## Requirements
- Operating System: Ubuntu 24.04 LTS (preferred) or Debian 12  
- Hardware:  
  - Minimum 2 vCPU  
  - 4GB RAM (8GB recommended)  
  - 60GB disk (for initial setup, expandable)  
- Internet access for package installation  
- Root or sudo-enabled user  

## Steps
1. **Boot the ISO**  
   - Select manual partitioning.  
   - Ensure UEFI boot if supported.  

2. **Partition Layout**
   - `/boot` → 2G, ext4, **non-LVM**  
   - Remaining disk → Physical Volume for LVM  
   - Create Volume Group `vg0` and Logical Volumes:  
     - `lv_root` → 30G, mount `/` (ext4)  
     - `lv_var` → 30G, mount `/var` (ext4)  
   - Leave free space for future LVs (like `/backup`).  

3. **Finalize installation**  
   - Set hostname, timezone, and a strong root password.  
   - Create a non-root user with sudo access.  

4. **Update base system**  
   - After first login, update packages and kernel.  

5. **Install essential packages**  
   - Editors, monitoring tools, networking utilities, etc.  

## Commands
```bash
# Update base system
sudo apt update && sudo apt upgrade -y

# Install essential tools
sudo apt install -y vim nano curl wget git htop ufw lvm2 \
  net-tools gnupg ca-certificates unzip tar bsdmainutils

# Enable unattended security upgrades
sudo apt install -y unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades

# Set hostname
sudo hostnamectl set-hostname linux-project-server
