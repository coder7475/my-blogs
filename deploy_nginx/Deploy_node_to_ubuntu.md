# How to Securely Deploy Node App to Ubuntu Server

This is documentation about how you can deploy a Node app to a server with Nginx, whether it is a `VPS`, `VDS`, or `dedicated server`. This assumes you're familiar with basic `Linux` & `git` commands. This will work for any Node app that runs a server, be it an `Express` app, `Next.js` app, `Remix` app, etc. Another thing to note is that we will deploy application code; the database will be separated.

## Assumptions

1. A hosting service with full root access and a domain
2. **OS**: Ubuntu 22.04
3. **IPv4**: `172.172.172.172`
4. **IPv6**: `2001:0db8:85a3:0000:0000:8a2e:0370:7334`
5. **Domain Name**: `example.com`
6. Using a `bash` terminal on your **local computer**
7. Code is hosted on `GitHub`

## Workflow Steps

### 1. Secure Your Server

---

#### A. Login to Your Server

1. Open your terminal (bash, zsh, etc.)

```
ssh root@172.172.172.172
```

You will be prompted to provide the root password. After entering it, you will be logged into your server. You should see something like this:

```bash
root@vm172048:~$
```

If so, you have successfully logged in as the root user.

#### B. Server Hardening

Now that we are inside our machine, we can start installing the necessary packages and software, but before that, let's upgrade our system. Enter the command below in your terminal:

```bash
apt update && apt upgrade -y
```

Now, all the current packages are updated to the latest patch version, which keeps the system safe from "unpatched vulnerability exploitation."

#### C. Create Non-Root User

Deploying as a root user is not recommended, as it has full access to all server resources. So let's create a **non-root user** called `admin` and add it to the `sudo` group to use commands that need root privileges.

To create a `sudo` user:

```bash
useradd -m -s /bin/bash admin
```

This will create a new user with the name `admin`, and you can check the groups of the user using the `groups` command.

```
groups admin
```

Let's add the `admin` user to the `sudo` group:

```bash
usermod -aG sudo admin
```

This will add the user to the `sudo` group without removing them from the original `admin` group.

Now let's **create a password** for the user:

```bash
sudo passwd admin
```

You will be prompted to enter a new password and retype it.

To **check if the password is set properly**, open a new terminal and check it by typing:

```
ssh admin@172.172.172.172
```

You will be prompted for your new password. After entering it, you should see something like:

```
admin@vm172048:~$
```

#### D. Connect to the Server Using SSH

Using **password login is not recommended**. You want to use SSH (Secure Shell) and make sure that **SSH is the only way to log in**.

If you are a user of `git`, chances are that you already have an `ssh key` set up. If you don't already have an `ssh key`, use the command below to generate a new SSH key on your **local machine**:

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

Follow the instructions; it should ask you where you want to save the file and if you want a passphrase or not. Just press the enter button for each prompt. Make sure you set a strong passphrase. To copy the public key over to your server, run the following command on your local machine:

```
ssh-copy-id -i ~/.ssh/id_ed25519.pub admin@172.172.172.172
```

Now, you should be able to log in without a password. If it still does not work, use `cat ~/.ssh/id_ed25519.pub` to copy the key. Then paste it into the server's `~/.ssh/authorized_keys`:

```bash
touch ~/.ssh/authorized_keys
sudo nano ~/.ssh/authorized_keys
```

After all of this, you should be able to log in without using a password.

#### E. Disable Root and Password Login on the Server

To turn off username and password login, type in:

```bash
sudo nano /etc/ssh/sshd_config
```

Find these values and set them as follows:

```bash
Port 1234 # Change the default port (use a number between 1024 and 65535)
PermitRootLogin no # Disable root login
PasswordAuthentication no # Disable password authentication
PubkeyAuthentication yes # Enable public key authentication
AuthorizedKeysFile .ssh/authorized_keys # Specify authorized_keys file location
AllowUsers admin # Only allow specific users to log in
```

This **disallows every login method besides SSH** under the user you copied your public key to. It stops login as root and only allows the user you specify to log in. Hit `CTRL+S` to save and `CTRL+X` to exit the file editor. Restart SSH:

```bash
sudo service ssh restart
```

Now try logging in as root to see if it disallows you. Since you changed the default SSH port from 22 to 1234, you need to mention the port when logging in.

```
ssh -p 1234 root@172.172.172.172
```

This will disallow you since root login is disabled. To log in, use:

```
ssh -p 1234 admin@172.172.172.172
```

Also, it should go without saying, but **you need to keep the private key safe**, and if you lose it, **you will not be able to get in remotely anymore**.

#### F. Firewall Configuration

Ubuntu comes with the `ufw` firewall by default. If not, you can install it with the command:

```
sudo apt install ufw -y
```

To see the current status of `ufw`, enter:

```
sudo ufw status
```

This will show the current status of the firewall. To enable the firewall, run the following command:

```
sudo ufw enable
```

First, run some default policies with:

```
sudo ufw default deny incoming && sudo ufw allow outgoing
```

Now, since we changed the SSH port to `1234`, allow it through the firewall. Aside from that, we will be using ports 80 and 443 for web serving via HTTP and HTTPS, so allow them too.

```
sudo ufw allow 1234,80,443
```

To further improve brute force login via SSH, use the command below:

```
sudo ufw limit 1234
```

This will limit port 1234 to 6 connections in 30 seconds from a single IP. If you opened any port incorrectly, you can deny the connection with:

```
sudo ufw deny <port_number>
```

Now, to see the current status, use:

```
sudo ufw status verbose
```

Restart the `ufw` to make sure all rules are applied:

```
sudo ufw reload
```

After enabling the firewall, **never** **exit from your remote server connection** without enabling the rule for the `ssh` connection. Otherwise, **you won't be able to log into your own server**.

#### G. Fail2Ban Configuration

Fail2Ban provides a protective shield for Ubuntu 22.04 that is specifically designed to block unauthorized access and brute-force attacks on essential services like [SSH and FTP](https://ultahost.com/blog/ssh-vs-ftp/). To install `fail2ban`, use the command below:

```
sudo apt install fail2ban
```

After the installation is complete, start the Fail2ban service with:

```
sudo systemctl start fail2ban
```

To **enable Fail2ban on Ubuntu 22.04** so that it starts automatically when your system boots up, use:

```
sudo systemctl enable fail2ban
```

Next, we need to verify if Fail2ban is up and running without any issues using the following command:

```
sudo systemctl status fail2ban
```

Now let's configure Fail2ban. The main configuration is located in `/etc/fail2ban/jail.conf`, but it's recommended to create a local configuration file:

```
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

Now open the `jail.local` file and tweak the values in the Default section:

```
bantime = 10m
findtime = 10m
maxretry = 5
```

Fail2Ban works by banning an IP for a specified **ban time** after detecting repeated failures within a defined **find time**. The **max retry** setting determines the number of failures allowed before an IP is banned.

Now restart the Fail2ban service:

```
sudo systemctl restart fail2ban
```

To see which service for which jail is activated, enter:

```
sudo fail2ban-client status
```

If you have come this far, congratulations! You have secured your server. Now it's ready to deploy your `webApp`. Enter `exit` to end the session.

```
exit
```

### 2. DNS Configuration

---

Our website will be known by a domain name, in this case, `example.com`. To make the domain name point to our server, we need to do some DNS configuration on the side of our domain provider.

a. Login to your domain provider's website.

b. Navigate to `example.com` and then manage DNS Management.

c. Now update the `A` and `AAAA` records for IPv4 & IPv6 Address.

| Record Type | Host Name | Address                                 |
| ----------- | --------- | --------------------------------------- |
| A           | @         | 172.172.172.172                         |
| AAAA        | @         | 2001:0db8:85a3:0000:0000:8a2e:0370:7334 |

d. Next, update the `CNAME` Record to forward `www.example.com` to `example.com`.

| Record Type | Host Name | Address     |
| ----------- | --------- | ----------- |
| CNAME       | www       | example.com |

`CNAME` maps a subdomain to another domain name.

Now it might take a few minutes to propagate to all DNS servers. To check if `example.com` resolves to your host `IP address`, check DNS propagation using this online tool: [DNS Checker](https://dnschecker.org/).

### 3. Deploy the Web App

---

To deploy, we will need to install several packages. First, we will be using `git` to clone the repository from `GitHub`. Since it's a Node app, we will need `Node` installed on our system; I will be using Node version 20. We will be using `npm`, which comes with `Node`. Finally, to manage the app as a background process, we will be using `pm2`.

First, log in to your server using:

```
ssh -p 1234 admin@172.172.172.172
```

#### A. Install All Necessary Dependencies

1. First, check the installation:

```
nginx -v
node -v
npm -v
git --version
pm2 --version
```

2. Then update to the latest version & remove unnecessary packages:

```
sudo apt update && sudo apt upgrade -y && sudo apt autoremove -y
```

3. Install the latest version of `git` available:

```
sudo apt install git -y
```

4. Install `Node` version 20 & its accompanying `npm` (Node Package Manager):

```
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo bash -
sudo apt-get install -y nodejs
```

5. Install the `nginx` web server:

```
sudo apt install -y nginx
```

#### B. Clone the Code Repo from GitHub

Our code repo will be hosted on `GitHub`. We will be cloning the repo to our server using a deploy key.

1. First, generate a deploy key named `site0`:

```
ssh-keygen -t ed25519 -C "fahad@octopusx.io" -f ~/.ssh/example
```

You will be prompted for a passphrase; just press enter. This will generate a public-private key pair called `example.pub` & `example` in your `~/.ssh` folder.

2. Make sure the `~/.ssh` folder is owned by `admin`:

```
sudo chown -R admin ~/.ssh
```

3. Add GitHub's SSH server public key to the server's `known_hosts` file:

```
ssh-keyscan -t ed25519 github.com >> ~/.ssh/known_hosts
```

4. Next, copy the SSH Public key after outputting the key to the terminal:

```
cat ~/.ssh/example.pub
```

5. Use the copied key as a deploy key in GitHub:

   - Go to your GitHub Repo
   - Click on the Settings Tab
   - Click on Deploy Keys option from the sidebar
   - Click on the Add Deploy Key Button and paste the copied SSH Public Key with a name of your choice
   - Click on Add Key

6. Clone the project from your GitHub Repo to your server's home using:

```
git clone git@github.com:admin7374/example_app.git
```

Here, `admin7374` is the GitHub username and `example_app` is the Node app we are about to deploy. This will clone the code repo on the server.

#### C. Run the App with pm2

Now it's time to build and run the Node app in the background:

1. Navigate to the project folder:

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

Example `.env` file:

```
PORT = 8001
DATABASE_URL = "database_url"
```

After pasting, click `CTRL + S` and `CTRL + X` to save and exit.

4. Create an `ecosystem.config.cjs` file in your repo code (best created inside the GitHub repo):

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
        port: 8001 // optional, if have port set in app
      }
  ]
}
```

The above code will run the Node app at port `8001`; make sure it matches the port defined in the application. The `script` is usually how the Node app generally runs. It assumes there is an `npm start` script inside your `package.json` to run the build code.

5. Next, install the necessary Node modules using:

```
npm ci
```

The above command creates a `node_modules` folder with all necessary packages to run the code.

6. Now, let's build the code. Type:

```
npm run build
```

The above script will build the code for distribution using the `build` script defined in `package.json`.

7. Add PM2 Process on Startup:

```
sudo pm2 startup
```

8. Start the Node App using `pm2`:

```
pm2 start ecosystem.config.cjs
```

9. Save the PM2 Process:

```
pm2 save
```

This will save the process to keep running in the background.

10. List all PM2 processes running in the background:

```
pm2 list
```

11. If you need to reload for redeployment, use:

```
pm2 reload example_app
```

12. To check the PM2 process logs, use:

```
pm2 monit
```

This will open an interactive terminal that will show you logs and metadata for each process. Enter `q` to quit.

13. Check if the app is working properly using:

```
curl localhost:8001
```

This should output properly if the app is working.

#### D. Serve the App with NGINX

Now it's time to finally serve the app using Nginx. Nginx is a powerful web server, reverse proxy, and load balancer. In this case, we will use the `nginx` reverse proxy feature to serve the app running on `localhost:8001` to the internet.

1. Start and enable `nginx`:

```
sudo systemctl start nginx
sudo systemctl enable nginx
```

2. Verify `nginx` is up and running:

```
sudo systemctl status nginx
```

If everything went well, the output should indicate that the Nginx service is `active (running)`.

3. If you wish to confirm Nginx's operation via a web browser, navigate to:

```
http://example.com
```

This should show the default `nginx` page.

4. If it's not showing, the `ufw` firewall may be blocking ports `80` and `443`. To allow them through the `ufw` firewall, use:

```
sudo ufw allow 'Nginx Full'
```

5. Nginx, like many server software, relies on configuration files to dictate its behavior. Begin by creating a configuration file for your website:

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

		# Proxy Params - pass client request information to the proxied server
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

		# If you need to upload files larger than 1M, use the below directive
        # client_max_body_size 500M;

        # Web socket upgrade configuration
        # Uncomment the following lines if you're using websockets in your app
        # proxy_http_version 1.1;
        # proxy_set_header Upgrade $http_upgrade;
        # proxy_set_header Connection "upgrade";
    }

	# Logging
	access_log /var/log/nginx/example.com.access.log;
	error_log /var/log/nginx/example.com.error.log warn;
}
```

The proxy pass configuration serves files directly; it proxies requests to a local application (in this case, running on port 8001).

7. With the configuration file created, it isn't live yet. To activate it, you'll create a symbolic link to the `sites-enabled` directory:

```
sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/example.com
```

Think of this step as "publishing" your configuration, making it live and ready to handle traffic.

8. Test the configuration before going live:

```
sudo nginx -t
```

Nginx will then parse your configurations and return feedback. A successful message indicates that your configurations are error-free.

9. Time to go live. This requires a reload:

```
sudo systemctl reload nginx
```

10. Now check in the web browser, go to:

```
http://example.com
```

11. Our current website configuration serves content over HTTP on port 80, which is unencrypted. Let's encrypt it via [Let's Encrypt](https://letsencrypt.org/). First, install certbot:

```
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository ppa:certbot/certbot
sudo apt update
sudo apt install certbot python3-certbot-nginx
```

12. Generate SSL Certificates using certbot:

```
sudo certbot --nginx -d example.com -d www.example.com
```

Just follow the instructions in the prompts. Your website will be encrypted with a proper SSL/TLS encryption certificate. Check your website in the browser to see if it's installed.

13. Nowadays, certbot, when getting a new cert, will set up auto-renew for you, so it's a sit-and-forget kind of task. But to make sure it worked, you can run:

```
sudo systemctl status certbot.timer
```

Now, big congratulations! You have successfully deployed your web app using Nginx. If you want to optimize Nginx, I recommend following this post: [Basic Nginx Setup](https://swissmade.host/en/blog/basic-nginx-setup-ubuntu-guide-to-a-functional-and-secure-website-serving).

This concludes my documentation on **how to deploy a Node app securely with Nginx**.

#### References - For More Information

1. [Server Setup & Hardening](https://docs.chaicode.com/server-setup/)
2. [Server Hardening Best Practices](https://perception-point.io/guides/os-isolation/os-hardening-10-best-practices/)
3. [Server Setup Basics](https://becomesovran.com/blog/server-setup-basics.html?ref=dailydev)
4. [Install fail2ban](https://ultahost.com/knowledge-base/install-fail2ban-ubuntu/)
5. [Fail2ban Ubuntu Configs](https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-20-04)
6. [Deploy Node.js to VPS](https://github.com/coder7475/GeekyShowsNotes/blob/main/nginx/Deploy_NodeExpress_Nginx.md)
7. [Nginx Configs](https://swissmade.host/en/blog/basic-nginx-setup-ubuntu-guide-to-a-functional-and-secure-website-serving)
8. [How to Change Default SSH Port](https://dev.to/coder7475/how-to-change-default-ssh-port-in-ubuntu-server-3aab)
9. [Uncomplicated Firewall](https://dev.to/coder7475/uncomplicated-firewall-ufw-1638)
10. [DNS Cheatsheet](https://dev.to/chrisachard/dns-record-crash-course-for-web-developers-35hn?ref=dailydev)
11. [SCP](https://www.linode.com/docs/guides/how-to-use-scp/)
12. [PM2 Guide](https://betterstack.com/community/guides/scaling-nodejs/pm2-guide/)
