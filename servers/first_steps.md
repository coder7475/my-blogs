## Server First 10 minutes for Ubuntu (Or all debian based Distros)

What to do after getting a server vps first time:

1. Check OS, Kernel, Hardware Architecture, Uptime

```bash
    cat /etc/ubuntu-advantage/uaclient.conf
    uname -a
    hostnamectl
    uptime
```

2. Who else logged In?

```bash
   who
   who -Hu
   grep sh$ /etc/passwd
```

3. Physical or Virtual Machine?

```bash
dmidecode -s system-manufacturer
dmidecode -s system-product-name
lshw -c system | grep product | head -1
cat /sys/class/dmi/id/product_name
cat /sys/class/dmi/id/sys_vendor
```

4. Hardware Information

- Check detailed information about CPU(s)

```bash
  cat /proc/cpuinfo
```

- Check systems memory usage

```bash
 cat /proc/meminfo
```

- Display all network interfaces

```bash
  ifconfig -a
  # OR
  ip addr
```

- Display settings and status of a network interface

```bash
  ethtool eth0
```

- List detailed hardware configuration

```bash
  lshw
```

- Display info about PCI buses and devices

```bash
  lspci
```

- All hardware info form system's DMI (desktop management interface)

```bash
 dmidecode
```

5. Check all Installed softwares

```bash
dpkg -l                          # Lists all installed packages
dpkg -l | grep <pkgname>         # Checks if a specific package is installed
apt-cache policy <pkgname>       # Displays information about a package (version, installation candidate, etc.)
apt list --installed             # Lists all installed packages using apt
apt list --installed | grep <pkgname>  # Checks if a specific package is installed using apt
apt-cache show <pkgname>         # Shows detailed information about a package
apt-cache madison <pkgname>      # Displays versions available in repositories
sudo apt update                  # Updates the list of available packages and their versions
apt-cache policy                 # Shows configured repositories and priorities
sudo apt install <pkgname>       # Installs a package
ls -l /etc/apt/sources.list.d/   # Lists all additional repository configurations
```

6. Running Processes & Service

```bash
  pstree -pa 1
  ps -ef
  ps auxf
  systemctl
```

7. Check Network Connections

```bash
  netstat -tulpn # Display a list of open network connection - ports, process
  netstat -anp   # Detailed info about all network connection
  lsof -i        # List open network file
  ss		 # Display socket statistics and network connections
  ss -t # Display all TCP sockets
  ss -u # Display all UDP Sockets
  ss -l # All listening sockets
  ss -a # All Sockets
  ss -s # Summary statistics
  ss -p # Process using the socket
  ss -n # Show numerical addresses instead of hostman
  iptables -L -n # List all firewall rules with IP address & port number
  cat /etc/resolv.conf # List info about DNS config of system
```

8. Kernel Information

```bash
  cat /boot/grub/grub.conf   # Displays the grub config file
  sysctl -a                  # Display all kernel parameter
  lsmod                      # Display status of modules currently loaded
  modinf <module>            # Display detailed information about a module
  uname -r                   # Display current kernel version
  cat /proc/cmdline          # Display all parameters passed to kernle at boot time

```

9. Logs

```bash
  journalctl               # Display logs collected by systemd-journald
  tail -f /var/log/syslog  # Monitor and display latest log entries
  demsg                    # Shows kernel's ring buffer messages
```

## References
