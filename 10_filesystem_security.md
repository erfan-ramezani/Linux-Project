# 10 – Filesystem Security (Append-only, Immutable)

## Requirements
- Create a file that allows only **append** operations (cannot edit or truncate).  
- Create another file that cannot be **modified or deleted** by anyone (even root), unless attribute is removed.  
- Apply on Debian-based systems (Debian 12 / Ubuntu 24.04).  

## Concepts
- Linux filesystems (ext4, xfs, …) support **extended attributes** using `chattr`.  
- Two important flags:  
  - `+a` → append-only (you can only add text, not overwrite or delete).  
  - `+i` → immutable (no modification, no deletion, no renaming, not even appending).  

## Steps
1. **Install required tools**  
   - `e2fsprogs` provides `chattr` and `lsattr` (preinstalled in Debian/Ubuntu).  

2. **Create an append-only file**  
   - Set `+a` flag so the file can only be appended.  

3. **Create an immutable file**  
   - Set `+i` flag so the file cannot be modified, deleted, renamed, or symlinked.  

4. **Test behavior**  
   - Try to edit, delete, or truncate → operation will fail.  
   - Only root can remove attributes with `chattr -a` or `chattr -i`.  

## Commands
```bash
# Ensure tools are installed (usually already present)
sudo apt update && sudo apt install -y e2fsprogs

# 1. Create append-only file
sudo touch /var/log/append_only.log
sudo chattr +a /var/log/append_only.log

# Try appending (works)
echo "New log entry" | sudo tee -a /var/log/append_only.log

# Try overwriting (fails)
echo "Overwrite attempt" | sudo tee /var/log/append_only.log

# Check attribute
lsattr /var/log/append_only.log
# Output should include: -----a--------


# 2. Create immutable file
sudo touch /important/immutable.conf
sudo chattr +i /important/immutable.conf

# Try editing (fails, even as root)
echo "Change" | sudo tee -a /important/immutable.conf
rm /important/immutable.conf  # will fail

# Check attribute
lsattr /important/immutable.conf
# Output should include: ----i---------


# 3. Remove attribute (only if necessary)
sudo chattr -a /var/log/append_only.log
sudo chattr -i /important/immutable.conf
