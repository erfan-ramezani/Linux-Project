# 08 – Monitoring & Restricted User

## Requirements
Create a restricted user (monitoring) with no root privileges
User can only run ls and cd
Home directory with limited permissions

## Steps
1.Create restricted user
   No sudo access
   Home directory with limited permissions
2.Configure restricted shell
   Use rbash to restrict available commands
3.Set allowed commands
   Only ls and cd are allowed
4.Restrict PATH and permissions
   Ensure the user cannot run other commands or access other directories  

## Commands
```bash
# Create restricted user
sudo adduser monitoring
sudo usermod -s /bin/rbash monitoring

# Create directory for allowed commands
sudo mkdir /home/monitoring/bin
sudo chown monitoring:monitoring /home/monitoring/bin
sudo ln -s /bin/ls /home/monitoring/bin/ls
# cd works as a shell builtin in rbash

# Restrict PATH and enable restricted mode
echo 'PATH=$HOME/bin' >> /home/monitoring/.bash_profile
echo 'export PATH' >> /home/monitoring/.bash_profile
echo 'set -r' >> /home/monitoring/.bash_profile

# Set home directory permissions
chmod 700 /home/monitoring
chmod 700 /home/monitoring/bin
