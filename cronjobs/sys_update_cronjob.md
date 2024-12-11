## How to set server hardening as Cronjob

### System update Cronjob

```bash
0 1 * * * sudo /home/admin/scripts/system_update.sh >> /var/log/system_update.log 2>&1
```

This cronjob runs daily at 1:00 AM and performs the following tasks:

- Updates package lists (`apt update`)
- Upgrades installed packages (`apt upgrade`)
- Removes unnecessary packages (`apt autoremove`)
- Logs all output to `/var/log/system_update.log`

**Prerequisites**:

1. Create a system update script and ensure the script has executable permissions in home folder of user `admin`:

   ```bash
   mkdir scripts
   touch system_update.sh
   chmod +x /home/admin/scripts/system_update.sh
   ```

2. Open and Add following code in `system_update.sh` script:

```bash
#!/bin/bash
# Script to update, upgrade, and autoremove packages

# Function to check if a command was successful
check_status() {
    if [ $? -ne 0 ]; then
        echo "Error: $1 failed"
        exit 1
    fi
}

echo "Starting system update and upgrade..."

# Run apt update
sudo apt update
check_status "apt update"

# Run apt upgrade with automatic yes to prompts
sudo apt upgrade -y
check_status "apt upgrade"

# Run apt autoremove to remove unnecessary packages
sudo apt autoremove -y
check_status "apt autoremove"

echo "System update, upgrade, and autoremove completed successfully."
```

3. Create the log file with appropriate permissions:

   ```bash
   sudo touch /var/log/system_update.log
   sudo chown admin:admin /var/log/system_update.log
   ```

4. Make sure the user has sudo privileges without password for these specific commands by adding the following to `/etc/sudoers` (use `sudo visudo` to open file):
   ```
   admin ALL=(ALL) NOPASSWD: /usr/bin/apt update, /usr/bin/apt upgrade -y, /usr/bin/apt autoremove -y
   ```

Note: Replace `admin` with your actual username if different.
