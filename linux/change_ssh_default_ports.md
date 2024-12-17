## Why Change SSH Port?

**Port 22** is the standard designated port for SSH connections.For enhanced security, it's highly recommended to change the default SSH port to a different, less obvious one. This makes it harder for attackers to target your SSH connection.

Here's why changing it is a smart security practice:

- **Brute-Force Attacks**: Automated scripts and bots constantly scan the internet for open port 22, trying to crack passwords with repeated login attempts (brute-force attacks). An unusual port number significantly reduces this risk.

- **Reduced "Noise"**: A standard SSH port receives constant connection attempts, many of them unauthorized. This generates unnecessary logs and can mask real attack attempts.

- **Security Through Obscurity**: It's one layer of defense (not a replacement for strong passwords or firewalls!). Attackers are less likely to spend time probing random ports.

- **Improved Organization**: If you manage multiple servers, using different SSH ports can help to identify and manage them more easily.

**Note**: Consider selecting a port outside the well-known range **(0-1023)** and the registered ports range **(1024-49151)**. It’s advisable to opt for a custom port within the dynamic or private ports range **(49152-65535)**.

## How to change default ssh port in vps

1. **Login** to your remote server using default port 22

```bash
  sudo ssh root@your_ip_address
```

Give password if asked.

2. **Backup**: Keeping a backup of your file is always a good option. Use this command to create a backup first:

```bash
 sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config_backup
```

3. **Change Port**: Open your **sshd_config** file using a editor:

```bash
  sudo vim /etc/ssh/sshd_config
```

Change commented out line from

```bash
  #Port 22
```

to port to your want to change

```bash
  Port 45673
```

save and exit

4. Restart the ssh service

```bash
  sudo service sshd restart
```

OR

```bash
  sudo systemctl restart sshd
```

5. Check if sshd service is restarted

```bash
  sudo systemctl status sshd
```

6. If your server has firewall enabled allow the server to listen on new port. For `ufw` use:

```bash
 sudo ufw allow 45673/tcp
```

7. Reload the firewall

```bash
  sudo ufw reload
```

8. Check the firewall status

```bash
  sudo ufw status
```

9. Now don't exit, open a new shell. Check if you can connect using new port:

```bash
  ssh -p 45673 root@your_ip_address
```

If you can, then your good to go. If it shows `refused to connect` then your firewall didn't allow the port, change the firewall rule. Or if it's show `Bad Port` then this port is used in other work, change the port.

Thanks for reading.

### References

1. [YouTube Video on Changing SSH Port](https://www.youtube.com/watch?v=bFgPpJs4ndQ&list=PLbGui_ZYuhij0mM8xP2udM_EDvl8JNdtn&index=13)

2. [Hostinger Tutorial on Changing SSH Port](https://www.hostinger.com/tutorials/how-to-change-ssh-port-vps)

3. [MonoVM Blog on Default SSH Port](https://monovm.com/blog/default-ssh-port/#:~:text=There%20are%20over%2065%2C000%20possible,designated%20port%20for%20SSH%20connections)
