# Installing-and-setting-up-nginx-with-Nodejs-and-MySQL

Installing and setting up Nginx with Node.js and MySQL involves several steps. Here's a comprehensive guide to get you started.

## Prerequisites

Ensure you have a Debian-based system with `sudo` privileges. In my case, Ubuntu 24.04.

## Step 1: Update and Upgrade Your System

```bash
sudo apt update
sudo apt upgrade -y
```

## Step 2: Install Nginx
Install Nginx using the package manager:
```bash
sudo apt install nginx -y
```
Start and enable Nginx to run on boot:
```bash
sudo systemctl start nginx
sudo systemctl enable nginx
```
Verify Nginx installation:
```bash
sudo systemctl status nginx
```

## Step 3: Install Node.js
Install Node.js:
```bash
sudo apt install -y nodejs
```
Verify the installation:
```bash
node -v

```
## Step 4: Install npm
install npm:
```bash
sudo apt install npm
```
Verify the installation:
```bash
npm -v
```

## Step 5: Git clone your repository and set up
Go to your localization on your system where you want to install your website, in my case it is `/var/www/`
```bash
cd /var/www/
sudo git clone https://github.com/your/link
```
`cd` into your cloned repository and Install node modules:
```bash
sudo npm i
```
### Install also pm2
```bash
sudo npm install pm2 -g
pm2 -v
```

## Step 6: Install MYSQL sever
Install mysql server:
```bash
sudo apt install mysql-server -y
```
Secure MySQL installation:
```bash
sudo mysql_secure_installation
```
During this process, you will be prompted to set a root password and remove some insecure default settings. Follow the prompts and choose the appropriate options for your setup.

Verify MySQL installation:
```bash
sudo systemctl status mysql
```

### configure MySQL to connect from different localization 
Open the MySQL configuration file (my.cnf or mysqld.cnf):
```bash
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```
Allow remote access:

Find the line that starts with `bind-address`. By default, it is set to `127.0.0.1`, which means MySQL will only accept connections from `localhost`.

Change this line to:
```bash
bind-address = 0.0.0.0
```
This will allow MySQL to accept connections from `any IP` address.
If you want you can put in your ip different so that none else can connect to your MySQL database

now you can save and exit with presing `Control + X` and `y` to save and enter

Restart MySQL service:
```bash
sudo systemctl restart mysql
```
### Create a MySQL User for Remote Access
Log in to MySQL as root:
```bash
sudo mysql -u root -p
```
Create a new user and grant remote access:

Replace remote_user, password, and remote_host_ip with your desired username, password, and the IP address of the remote host. Use % as remote_host_ip if you want to allow access from any IP address.

```bash
CREATE USER 'remote_user'@'remote_host_ip' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON *.* TO 'remote_user'@'remote_host_ip' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```
For example, to allow access from any IP address:
```bash
CREATE USER 'remote_user'@'%' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON *.* TO 'remote_user'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```
Exit MySQL:
```bash
EXIT;
```
Step 7: Adjust Firewall Settings
Open MySQL port in the firewall:

MySQL uses port 3306 by default. Use the following commands to open this port:
```bash
sudo ufw allow 3306/tcp
```
open port for Nginx:
```bash
sudo ufw allow 'Nginx Full'
```
open port for ssh
```bash
sudo ufw allow ssh
```
open port for http and https
```bash
sudo ufw allow http
sudo ufw allow https
```
open port for default dns port
```bash
sudo ufw allow 53
```
open port for your nodejs aplication (probobly 3000):
```bash
sudo ufw allow 3000
```
Enable, disable, and restart commands ufw:
```bash
sudo ufw enable
sudo ufw disable
sudo ufw reset
```
## Step 8: Configure Nginx for a Node.js App
Create a new Nginx server block file:
```bash
sudo nano /etc/nginx/sites-available/your_domain
```
Here paste in this code and edit on `server_name`, 
Change on `proxy_pass` port your app is runing at to port(Most probobly 3000).
Change on localization of where your files are (It is most comon to put them in `/var/www/your_githug_reposetory`)
```bash
server {
    listen 80;
    listen [::]:80;
    server_name your_domain.com;

    location / {
        proxy_pass http://localhost:(your_port);
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    location /views/ {
        internal;
        alias /your/localization/views/;
    }

    location /public/ {
        alias /your/localization/My-Portfolio-webside/public/;
    }

    location ~ \.ejs$ {
        try_files $uri $uri/ @nodejs;
    }
    
    location @nodejs {
        proxy_pass http://localhost:(your_port);
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```
Exemple:

```bash
server {
    listen 80;
    listen [::]:80;
    server_name your_domain.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    location /views/ {
        internal;
        alias /var/www/MyWebside/My-Portfolio-webside/views/;
    }

    location /public/ {
        alias /var/www/MyWebside/My-Portfolio-webside/public/;
    }

    location ~ \.ejs$ {
        try_files $uri $uri/ @nodejs;
    }
    
    location @nodejs {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```
### Unlin Defoult and link your website

Remove the symlink for the default server block:
```bash
sudo unlink /etc/nginx/sites-enabled/default
```
Create the symlink:
```bash
sudo ln -s /etc/nginx/sites-available/your_domain /etc/nginx/sites-enabled/
```

Test the Nginx configuration for syntax errors:
```bash
sudo nginx -t
```
This command checks the configuration files for any syntax errors. You should see output similar to:

`nginx: the configuration file /etc/nginx/nginx.conf syntax is ok`

`nginx: configuration file /etc/nginx/nginx.conf test is successful`

Reload the Nginx service to apply the changes:
```bash
sudo systemctl reload nginx
```







