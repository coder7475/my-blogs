# How to Securely Deploy Node App to Ubuntu Server

This is documentation about how you can deploy a node app to a server with nginx whether it is a `vps`, `vds`, `dedicated server` instance etc. This assumes your familiar with basic `linux` & `git ` commands. This will work for any node app that runs a server be it express app, next.js app, remix app etc. Another thing to note that we will deploy application code, database will be separated.

## Assumptions

1. A hosting with full root access and a domain
2. **OS**: Ubuntu 22.04
3. **IPv4**: `172.172.172.172`
4. **IPv6**: `2001:0db8:85a3:0000:0000:8a2e:0370:7334`
5. **Domain Name**: `example.com`
6. Using a `bash` terminal on your **local computer**
7. Code is hosted on `GitHub`

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

Using **password to login is not recommended**. You want to use SSH (Secure Shell) and make sure that **SSH is the only way to log in**.

If you are a user of `git` chance is that you already have a `ssh key` setup. If you don't already have a `ssh key` use below command to generate a new ssh key in your **local machine:**

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

Follow the instructions, it should ask you where you want to save the file and if you want a passkey or not. Just click enter button for each prompt. Make sure you set a string one. To copy the public key over to your server run on your local machine:

```
ssh-copy-id -i ~/.ssh/id_ed25519.pub admin@172.172.172.172
```

Now, you should be able to login without a password. If it still does not work. Then use `cat ~/.ssh/id_ed25519.pub` then copy the copied key. Then paste it into **server's** `~/.ssh/authorized_keys`:

```bash
touch ~/.ssh/authorized_keys
sudo nano ~/.ssh/authorized_keys
```

After all of this you should be able to log in without using password.

#### E. Disable root and password login In the Server

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
```

This would disallow you as root login is disabled. To login use:

```
ssh -p 1234 admin@172.172.172.172
```

Also, it should go without saying, but **you need to keep the private key safe** and if you lose it **you will not be able to get in remotely anymore**.

#### E. Firewall Configuration

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

Now open the `jail.local` file, Tweak the values in Default section:

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

To see which service for which jail is activated, enter:

```
sudo fail2ban-client status
```

If you have come so far. Congratulations. You have secured your server. Now it's ready to deploy your `webApp`. Enter `exit` to end the session.

```
exit
```

### 2. DNS Configuration

---

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

Now It might take a few minutes to propagate to all DNS servers. To check if `example.com` resolve to you host `ip address`. Check DNS Propagation by this online tool: [DNS Checker](https://dnschecker.org/)

### 3. Deploy the Web App

---

To deploy, we will need to install several packages, first we will be using `git` to clone the repository from `github`. Since it's a node app we will need `node` installed in our system, I will be using node version 20. We will be using `npm` which comes with `node`. Finally to manage the app as background process, we will be using `pm2`.

First Login to your server using:

```
ssh -p 1234 admin@172.172.172.172
```

#### A. Install all Necessary Dependencies

1. First, To check the installation:

```
nginx -v
node -v
npm -v
git --version
pm2 --version
```

2.  Then update to latest version & remove unnecessary packages:

```
sudo apt update && sudo apt upgrade -y && sudo apt autoremove -y
```

4. Install latest version of `git` available:

```
sudo apt install git -y
```

5. Install `node` version 20 & its accompanying `npm` (Node Package Manager):

```
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo bash -
sudo apt-get install -y nodejs
```

6. Install `nginx` web server:

```
sudo apt install -y nginx
```

#### B. Clone the Code Repo from GitHub

Our code repo will be hosted on `github`. We will be cloning the repo to our server using deploy key.

1. First generate a deploy key name `site0`:

```
ssh-keygen -t ed25519 -C "fahad@octopusx.io" -f ~/.ssh/example
```

You will be prompted for passkey just click enter. This will generate a public-private key pair called `example.pub` & `example` in your `~/.ssh` folder 2. Make sure the `~/.ssh` folder is owned by `admin`

```
sudo chown -R admin .ssh
```

3. Add GitHub's SSH server public key to server's `known_host` file:

```
ssh-keyscan -t ed25519 github.com >> ~/.ssh/known_hosts
```

5. Next Copy the SSH Public key after outputting the key to terminal:

```
cat ~/.ssh/example.pub
```

6. Use the copied key as deploy key in GitHub:
   - Go to Your Github Repo
   - Click on Settings Tab
   - Click on Deploy Keys option from sidebar
   - Click on Add Deploy Key Button and Paste Copied SSH Public Key with a name of your choice
   - Click on Add Key
7. Clone Project from your github Repo to your server's home using:

```
git clone git@github.com:admin7374/example_app.git
```

here, `admin7374` is the github username and `example_app` is the node app we are about to deploy. This will clone the code repo in server.

#### C. Run the App with pm2

Now it's time to build and run the node app in the background:

1. Navigate to project folder

```
cd ~/example_app
```

2. Create a `.env` file:

```
touch .env
```

3. Open the `.env` file and paste your environmental variables:

```
sudo nano .env
```

example `.env` file:

```
PORT = 8001
DATABASE_URL = "database_url"
```

After pasting click `ctrl + s` and `ctrl + x` to save and exit. 4. Create a `ecosystem.config.cjs` in your repo code (best created inside github repo):

```
touch ecosystem.config.cjs
sudo nano ecosystem.config.cjs
```

Then paste the code below:

```
module.exports = {
  apps : [
      {
        name: "example_app",
        script: "npm start",
        port: 8001
      }
  ]
}
```

Above code will run the node app at port `8001` make sure it matches your port defined in application. The `script` is usually how the node app generally runs. It assumes there is `npm start` script inside your `package.json` the run the build code. 5. Next install necessary node module using:

```
npm ci
```

above command create a `node_modules` folder with all necessary packages to run the code. 7. Now, lets build the code, type:

```
npm run build
```

Above script will build the code for distribution using the `build` script defined in `pacakge.json` 7. Add PM2 Process on Startup:

```
sudo pm2 startup
```

8. Start the Node App using `pm2`

```
pm2 start ecosystem.config.cjs
```

9. Save PM2 Process

```
pm2 save
```

This will save the process to keep running in the background. 10. List all pm2 process running in the background:

```
pm2 list
```

11. If you need to reload for redeployment, use:

```
pm2 reload example_app
```

12. To check the pm2 process logs use:

```
pm2 monit
```

This will open a interactive terminal which will show you logs and metadata for each process. Enter `q` to quit. 13. Check if the apps working properly using:

```
curl localhost:8001
```

This will output properly if the app is working.

#### D. Serve the App with NGINX

Now it's time to finally serve the app to using nginx. Nginx is a powerful web server, reverse proxy and load balancer. In This case we will using the `nginx` reverse proxy feature to server the app running on `localhost:8001` to internet.

1. Start and Enable `nginx`

```
sudo systemctl start nginx
sudo systemctl enable nginx
```

2. Verify `nginx` is up and running:

```
sudo systemctl status nginx
```

If everything went well, the output should indicate that the Nginx service is `active (running)`. 3. If you wish to confirm Nginx's operation via a `web browser`, then in navigate to:

```
http://example.com
```

This will show default `nginx` page. 4. If it's not showing, the `ufw` firewall is blocking port `80` and port `443`. To allow through `ufw` firewall use:

```
sudo ufw allow 'Nginx Full'
```

5.  Nginx, like many server software, relies on configuration files to dictate its behavior. Begin by creating a configuration file for your website

```
sudo nano /etc/nginx/sites-available/example.com
```

6. Inside this file, input the following proxy pass configuration:

```
server {
    listen 80;
    listen [::]:80;
    server_name example.com www.example.com;

    location / {
        proxy_pass http://localhost:8001;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        client_max_body_size 500M;
    }

	# Logging
	access_log /var/log/nginx/example.com.access.log;
	error_log /var/log/nginx/example.com.error.log warn;
}
```

The proxy pass configuration serves files directly, it proxies requests to a local application (in this case, running on port 8001). 7. With the configuration file created, it isn't live yet. To activate it, you'll create a symbolic link to the `sites-enabled` directory:

```
sudo ln -s /etc/nginx/sites-available/mywebsite /etc/nginx/sites-enabled/
```

Think of this step as "publishing" your configuration, making it live and ready to handle traffic. 8. Test the configuration before going live:

```
sudo nginx -t
```

Nginx will then parse your configurations and return feedback. A successful message indicates that your configurations are error-free. 9. Time to go live. This require a reload:

```
sudo systemctl reload nginx
```

10. Now check in the web browser, go to:

```
http://example.com
```

11. Our current website configuration serves content over HTTP on port 80, which is unencrypted. Let's encrypt it via [Let's Encrypt](https://letsencrypt.org/). First install certbot:

```
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository ppa:certbot/certbot
sudo apt update
sudo apt install certbot python3-certbot-nginx
```

13. Generate SSL Certificates using certbot:

```
sudo certbot --nginx -d example.com -d www.example.com
```

Just follow the instruction in the prompts. You website will be encrypted with proper ssl/tls encryption certificate. Check your website on browser to see if it's install. 14. Nowadays certbot when getting a new cert will set up auto-renew for you, so it's a sit-and-forget kinda task. But to make sure it worked you can run:

```
sudo systemctl status certbot.timer
```

Now, big congratulation you have successfully deployed your web app using nginx. If you want to optimize nginx, I recommend following this post: [Basic Nginx Setup](https://swissmade.host/en/blog/basic-nginx-setup-ubuntu-guide-to-a-functional-and-secure-website-serving)

So this end the my documentation about how to deploy a node app with nginx. I will follow up with a documentation about how to automate the deployment with CI/CD pipeline in next Documentation.

#### References

1. [Server Setup & Hardening](https://docs.chaicode.com/server-setup/)
2. [Server Hardening Best Practices](https://perception-point.io/guides/os-isolation/os-hardening-10-best-practices/)
3. [Server Setup Basices](https://becomesovran.com/blog/server-setup-basics.html?ref=dailydev)
4. [Install fail2ban](https://ultahost.com/knowledge-base/install-fail2ban-ubuntu/)
5. [fail2ban ubuntu configs](https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-20-04)
6. [Deploy Node.js to VPS](https://github.com/coder7475/GeekyShowsNotes/blob/main/nginx/Deploy_NodeExpress_Nginx.md)
7. [Nginx Configs](https://swissmade.host/en/blog/basic-nginx-setup-ubuntu-guide-to-a-functional-and-secure-website-serving)
8. [How to change default ssh port](https://dev.to/coder7475/how-to-change-default-ssh-port-in-ubuntu-server-3aab)
9. [Uncomplicated Firewall](https://dev.to/coder7475/uncomplicated-firewall-ufw-1638)
10. [DNS Cheatsheet](https://dev.to/chrisachard/dns-record-crash-course-for-web-developers-35hn?ref=dailydev)
11. [scp](https://www.linode.com/docs/guides/how-to-use-scp/)
12. [pm2 guide](https://betterstack.com/community/guides/scaling-nodejs/pm2-guide/)
