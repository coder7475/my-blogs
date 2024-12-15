# General Commands

**See Ubuntu OS version**:

```bash
  lsb_release -a
```

**List the users of the OS**:

```bash
   who
```

**Find the current user**:

```bash
  whoami
```

**See the uptime of the server**:

```bash
uptime
```

**Change the computer hostname in UBUNTU**:

1. Access the hostname file

```bash
  sudo nano /etc/hostname
```

2. Edit the file change the host name:

```bash
  sudo nano /etc/hosts
```

3. Reboot the system

```bash
  sudo reboot
```

4. Make a file undeletable

```bash
   sudo chattr +i -V filename.txt
```

5. Remove a directory with its subdirectories and files

```bash
  yes "yes" | rm -vRI directory
```
