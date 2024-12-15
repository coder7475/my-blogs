## Linux Firewalls

- All modern Linux firewall solution uses **Netfilter** subsystem.

- **Netfilter** is a packet filtering system that is used to
  manipulate the fate of network traffic headed into or through the server.

- _System administrator_ use userspace interface utility _siptables_ to set rules for how to manage the incoming traffic.

- _iptables_ is extremely effective and customizable, but it can be complex to configure.

- Developers produced several frontend to help user control their firewall without writing lengthy _iptables_ rules. Ex: ufw, firewalld etc

## ufw - Uncomplicated Firewall

- The default for debian based distros, ex: ubuntu, linux mint etc.

- Provides a user-friendly way to create IPv4 or IPv6 host-based firewall.

- `ufw` by default is initially disabled.

### Enable or disable ufw

To enable `ufw`, run:

```bash
  sudo ufw enable
```

To disable `ufw`, run:

```bash
  sudo ufw disable
```

### Check the status

To see the firewall status, enter:

```bash
  sudo ufw status
```

See numbered format:

```bash
  sudo ufw status numbered
```

Show all added rules:

```bash
ufw show added
```

### UFW Defaults

It's very important to understand ufw defaults for your security.

Enter:

```bash
  sudo ufw status verbose
```

Above command will result:

```bash
# ufw status verbose
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip
To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere
44                         DENY IN     Anywhere
22/tcp (v6)                ALLOW IN    Anywhere (v6)
44 (v6)                    DENY IN     Anywhere (v6)
```

Explanation of output below:

1. `deny (incoming)`: This will make sure that no outside systems can connect to your machine until you add an overriding rule for it.

2. `allow (outgoing)`: This means that all outgoing requests are enabled. This setting helps you run commands like apt-install, wget, and ping without issues. But, if you want to keep your server secure it is better to change the defaults to block outgoing and then allow specific IPs/domains that you need.

3. `disabled (routed)`. This means that all routing is disabled and forwarding is blocked. This is a good default provided you are not using your machine as a router.

4. In **Action** column it is `ALLOW IN` & `DENY IN`. Which means there is also `ALLOW OUT` & `DENY OUT`.

## Reload firewall for new rules

If UFW is already enabled and you modify the firewall rules, you need to reload it before the changes take into effect.

You can restart UFW by disabling it and enabling it again:

```bash
sudo ufw disable && sudo ufw enable
```

Or reload the rules:

```bash
sudo ufw reload
```

### Reset all rules of ufw

```bash
  ufw reset
```

### How to add ufw rules

Syntax to add rule:

```bash
sudo ufw allow <port>/<optional: protocol>
sudo ufw deny <port>/<optional: protocol>
```

#### Examples

1. To open a port (port no: 22):

```bash
  sudo ufw allow 22
```

2. To close an opened port:

```bash
  sudo ufw deny 22
```

3. To allow ssh connection

```bash
  ufw allow ssh
```

4. To allow http and https

```bash
  sudo ufw allow http && sudo ufw allow https
```

_Rules can also be added using a \_numbered_ format.\_

5. See numbered format:

```bash
  sudo ufw status numbered
```

6. To add a rule using _numbered_ format:

```bash
  sudo ufw insert 1 allow 80
```

This allowing 80 port as number 1 rule

7. To remove a rule, use `delete` followed by the rule:

```bash
  sudo ufw delete deny 22
```

This delete the deny 22 rule

### To check all open ports that are running

1. Install `net-tools` if not already installed

```bash
  sudo apt install net-tools
```

2. Show all open port that are currently running:

```bash
  netstat -tulpn
```

3. To further check your network connection use:

```bash
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

## Allow Access from specific hosts

- It can allow access from specific hosts or networks to a port

- Example: Allows SSH access from host `192.168.0.2` to any IP address on this host:

```bash
 sudo ufw allow proto tcp from 192.168.0.2 to any port 22
```

- To allow SSH access from entire subnet enter:

```bash
 sudo ufw allow proto tcp from 192.168.0.2/24 to any port 22
```

### Simulate Adding Rules

If you want to see what happens when you add a rule use `--dry-run` option to a `ufw` command.

```bash
  sudo ufw --dry-run allow http
```

### Configure to support IPv6

1. Open Config File: using nano(a text editor)

```bash
  sudo nano /etc/default/ufw
```

2. Then Change The IPV6 value to yes:

```bash
 IPV6=yes
```

### ufw application integration

1. See all available apps:

```bash
  sudo ufw app list
```

2. Syntax to add or deny app:

```bash
sudo ufw allow <application>
sudo ufw deny <application>
```

3. To allow `OpenSSH` enter:

```bash
  sudo ufw allow "OpenSSH"
```

### Special Tips For Newbies

- After enabling firewall **never** exit from your remote server connection without `enabling` rule for `ssh` connection. Otherwise you won't be able to log into your own server.

## UFW Logging

1. To see if logging is enabled:

```bash
  sudo ufw status verbose
```

2. To allow logging on:

```bash
  sudo ufw logging on
```

### Different levels of UFW Firewall logging

There are 5 levels of UFW logging.

1. `off`: Means logging is disabled.
2. `low`: Will store logs related to blocked packets that do not match the current firewall rules and will show log entries related to logged rules.
3. `medium`: In addition to all the logs offered by the low level, you get logs for invalid packets, new connections, and logging done through rate limiting.
4. `high`: Will include logs for packets with rate limiting and without rate limiting.
5. `full`: This level is similar to the high level but does not include the rate limiting.

### To change logging level

1. Syntax

```bash
  sudo ufw logging logging_level
```

2. If you want to change it to medium level

```bash
  sudo ufw logging logging_level
```

### Check logs

1. See the Full logs:

```bash
  sudo less /var/log/ufw.log
```

2. See only last 10 line of log

```bash
  sudo tail -f /var/log/ufw.log
```

### References

- [Ubuntu Server Docs: Firewalls](https://ubuntu.com/server/docs/firewalls)
- [OpenSource.com: Firewall Cheat Sheet](https://opensource.com/article/20/2/firewall-cheat-sheet?ref=dailydev)
- [GeekyShowsNotes: UFW Firewall Setup](https://github.com/coder7475/GeekyShowsNotes/blob/main/ufw_firewall_setup.md)
- [DigitalOcean: UFW Essentials - Common Firewall Rules and Commands](https://www.digitalocean.com/community/tutorials/ufw-essentials-common-firewall-rules-and-commands)
- [Better Programming: Understanding UFW](https://betterprogramming.pub/understanding-ufw-8d70d5d8f9d2)
- [It's FOSS: UFW on Ubuntu](https://itsfoss.com/ufw-ubuntu/?ref=dailydev)
- [OpenSource.com: Linux Server](https://opensource.com/article/20/12/linux-server?ref=dailydev)
- [DigitalOcean: How to Set Up a Firewall with UFW on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu?ref=dailydev)
