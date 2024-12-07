## How to Deploy to Ubuntu Server

This is documentation about how you can deploy a node app to a server with nginx whether it is a `vps`, `vds`, `dedicated server`, `aws ec2` instance etc. This assumes your familiar with basic linux commands.

### Assumptions

1. A hosting with full root access and a domain
2. **OS**: Ubuntu 22.04
3. **IP**: 172.172.172.172
4. **Domain Name**: example.com
5. Using a Linux terminal on your local computer

### Workflow Steps

---

#### 1. Secure Your Server

##### A. Login to your server

1. Open your terminal (bash, zsh etc)

```
ssh root@172.172.172.172
```

you will be prompted to give root password. After giving you will be logged into your server. See something like this:

```bash
root@vm172048:~$
```

If so you have Successfully logged in as root user.

##### B. First Server Hardening

Now we are inside our machine and we can start installing the necessary packages and software but before that let’s upgrade our system. Enter below command in your terminal:

```bash
apt update && apt upgrade -y
```

Now, all the current packages are installed to latest patch version which keeps the system safe from "unpatched vulnerability exploitation"
