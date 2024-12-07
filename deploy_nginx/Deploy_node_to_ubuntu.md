# How to Securely Deploy Node App to Ubuntu Server

This is documentation about how you can deploy a node app to a server with nginx whether it is a `vps`, `vds`, `dedicated server`, `aws ec2` instance etc. This assumes your familiar with basic linux commands. This will work for any node app that runs a server be it express app, next.js app, remix app etc.

## Assumptions

1. A hosting with full root access and a domain
2. **OS**: Ubuntu 22.04
3. **IPv4**: 172.172.172.172
4. **IPv6**: 2001:0db8:85a3:0000:0000:8a2e:0370:7334
5. **Domain Name**: example.com
6. Using a Linux terminal on your local computer

## Workflow Steps

---

### 1. Secure Your Server

---

#### A. Login to your server

1. Open your terminal (bash, zsh etc)

```
ssh root@172.172.172.172
```

you will be prompted to give root password. After giving you will be logged into your server. See something like this:

```bash
root@vm172048:~$
```

If so you have Successfully logged in as root user.

#### B. Server Hardening

Now we are inside our machine and we can start installing the necessary packages and software but before that let’s upgrade our system. Enter below command in your terminal:

```bash
apt update && apt upgrade -y
```

Now, all the current packages are installed to latest patch version which keeps the system safe from "unpatched vulnerability exploitation"

#### C. Create Non-Root User

---

Deploying as a root user is not recommended as it has full access to server all resources. So let's create a **not-root user** called `admin` and add it to to `sudo` group to use commands that need root previlieges.

For creating a `sudo` user:

```bash
useradd -m -s /bin/bash admin
```

This will create a new user with the name `admin` and you can check the groups of the user using the `groups` command.

```
groups admin
```

Let's add the `admin` user to `sudo` group:

```bash
usermod -aG sudo admin
```

This will add the user to `sudo` group without removing from original `admin` group.

Now lets **create a password** for the user:

```bash
sudo passwd admin
```

you will be prompted to give new password and retype it.

So **check if password is set properly**, open a new terminal check it by typing:

```
ssh admin@172.172.172.172
```

You will be prompted for your new password. After giving you will see something like:

```
admin@vm172048:~$
```

#### D. Connect to the server Using SSH

---

Using **password to login is not recommended**. You want to use SSH (Secure Shell) and make sure that **SSH is the only way to log in**.

If you are a user of `git` change is you already have a `ssh key` serup. If you don't already have a `ssh key` use below command to generate a new ssh key:

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

Follow the instructions, it should ask you where you want to save the file and if you want a password or not. Make sure you set a string one. To copy the public key over to your server run on your local machine:

```
ssh-copy-id -i ~/.ssh/id_ed25519.pub admin@172.172.172.172
```

Now, you should be able to login without a password. If it still does not work. Then use `cat ~/.ssh/id_ed25519.pub` then copy the copied key. Then paste it into `~/.ssh/authorized_keys`:

```bash
touch ~/.ssh/authorized_keys
sudo nano ~/.ssh/authorized_keys
```

After all of this you should be able to log in without using password.

#### E. Disable root and password login In the Server

---

To turn off username and password login, type in:

```bash
sudo nano /etc/ssh/sshd_config
```

Find this value and set as in:

```bash
Port 1234 # Change the defaul port (use a number between 1024 and 65535)
PermitRootLogin no # Disable root login
PasswordAuthentication no # Disable password authentication
PubkeyAuthentication yes # Enable public key authentication
AuthorizedKeysFile .ssh/authorized_keys # Specify authorized_keys file location
AllowUsers admin # Only allow specific users to login
```

This **disallows every login method besides SSH** under the user you copied your public key to. Stops login as Root and only allows the user you specify to log in. Hit CTL+S to save and CTL+X to get out of the file editor. Restart SSH:

```bash
sudo service ssh restart
```

Now try login as root to see if disallows you. Since you changed default ssh key port 22 to 2222 Then you need to mention port when login.

```
ssh -p 1234 root@172.172.172.172
ssh -p 1234 admin@172.172.172.172
```

Also, it should go without saying, but **you need to keep the private key safe** and if you lose it **you will not be able to get in remotely anymore**.

#### E. Firewall Configuration

---

Ubuntu comes with `ufw` firewall by default. If not you can install by command:

```
sudo apt install update && sudo apt install ufw -y
```

So see the current status of `ufw` enter:

```
sudo ufw status
```

This will show the current status of the firewall. To enable the firewall, run the following command:

```
sudo ufw enable
```

First run some default policies with:

```
sudo ufw default deny incoming && sudo ufw allow outgoing
```

Now since we change ssh port to `1234` allow it via firewall. Aside for them we will be using port 80 & port 443 for web serving via http & https so allow them too.

```
sudo ufw allow 1234, 80, 443
```

To further improve brute force login via ssh use below command:

```
sudo ufw limit 1234
```

This would limit port 1234 to 6 connections in 30 seconds from a single IP.
If you opened any port wrongly you can deny connection with:

```
sudo ufw deny <port_number>
```

Now to see current status use:

```
sudo ufw status verbose
```

Restart the `ufw` to make sure all rules are applied:

```
sudo ufw reload
```

After enabling firewall **never** **exit from your remote server connection** without `enabling` rule for `ssh` connection. Otherwise **you won't be able to log into your own server**.

#### F. Fail2Ban Configuration

---

Fail2Ban provides a protective shield for Ubuntu 22.04 that is specifically designed to block unauthorized access and brute-force attacks on essential services like [SSH and FTP](https://ultahost.com/blog/ssh-vs-ftp/).
To install `fail2ban` use below command:

```
sudo apt install fail2ban
```

After the installation is complete, start the Fail2ban service with:

```
sudo systemctl start fail2ban
```

To **enable Fail2ban on Ubuntu 22.04** so that it starts automatically when your system boots up, we can use:

```
sudo systemctl enable fail2ban
```

Next, we need to verify if Fail2ban is up and running without any issues using the following command:

```
sudo systemctl status fail2ban
```

Now lets, configure fail2ban. The main configuration is located in /etc/fail2ban/jail.conf, but it's recommended to create a local configuration file:

```
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

Now open the `jail.local` file:

```
bantime = 10m
findtime = 10m
maxretry = 5
```

Fail2Ban works by banning an IP for a specified **ban time** after detecting repeated failures within a defined **find time**. The **max retry** setting determines the number of failures allowed before an IP is banned.

Now restart the fail2ban service:

```
sudo systemctl restart fail2ban
```

If you have come so far. Congratulations. You have secured your server. Now it's ready to deploy your `webApp`. Enter `exit` to end the session.

```
exit
```

### 2. DNS Configuration

Our website will known by a domain name, in this case `example.com`. To make the domain name point to our server we need to some DNS configuration on the site of our domain provider.

a. Login to your domain providers website

b Navigate to `example.com` and then Manage DNS Management

c. Now Update `A` and `AAAA` record for IPv4 & IPv6 Address

| Record Type | Host Name | Address                                 |
| ----------- | --------- | --------------------------------------- |
| A           | @         | 172.172.172.172                         |
| AAAA        | @         | 2001:0db8:85a3:0000:0000:8a2e:0370:7334 |

d. Next Update the `CNAME` Record to forward `www.example.com` to `example.com`

| Record Type | Host Name | Address     |
| ----------- | --------- | ----------- |
| CNAME       | www       | example.com |

`CNAME` maps a subdomain to another domain name

Now It might take a few minutes to propagate to all DNS servers. To check if `example.com` resolve to you host ip address. Check DNS Propagation by this online tool: [DNS Checker](https://dnschecker.org/)
