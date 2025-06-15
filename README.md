## How I Deployed the Standcore Project

## 1️⃣ Server Provisioning on AWS

Logged into AWS and launched an EC2 instance (region: Ireland).

Chose Ubuntu, created a key pair, and allowed ports 22, 80, and 443.

Allocated 20 GB of gp2 storage.

Connected to the server from a Linux terminal using SSH:

Connect to your Server
   ```bash
   ssh -i "Ire-key-2.pem" ubuntu@ec2-xxx-xxx-xxx-xxx.eu-west-1.compute.amazonaws.com
   ```

---

##  2️⃣ Initial Server Setup

1. **Updated the server and installed Node.js and npm**
   ```bash
   sudo apt update
   sudo apt install nodejs npm -y
   ```

2. **Verified installations**
   ```bash
   node -v
   npm -v
   ```
   
## 3️⃣ Connecting my public IP address with a custom domain name

* Open the dashboard of the domain name vendor, in my case https://freedns.afraid.org/subdomain/edit.php
* Add new subdomain name
* Edit the A record to my public IP address from my server on AWS
* Save and exit(Note that it takes some times for the dns to resolve)
---

## 4️⃣ Project Structure and Development

1. **Created the project directory:**
   ```bash
   mkdir Standcore && cd Standcore
   ```

2. **Initialized npm and installed Express:**
   ```bash
   npm init -y
   npm install express
   ```

3. **Project structure**
   ```bash
   Standcore/
   public/
   index.html
   styles.css
   scripts.js
   server.js
   ```

---
4. **Server setup (server.js)**
```bash
const express = require('express');
const path = require('path');
const app = express();

app.use(express.static(path.join(__dirname, 'public')));

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Standcore running on http://localhost:${PORT}`);
});
```
5. **Ran the server**
```bash
 node server.js
```

##  5️⃣ Run your Node App with a Process Manager

1. **Install PM2**
   ```bash
   sudo npm install -g pm2
   pm2 start server.js
   pm2 startup
   ```

2. **Copy and run the command PM2 suggests (e.g.)**
   ```bash
   sudo env PATH=$PATH:/usr/bin pm2 startup systemd -u ubuntu --hp /home/ubuntu
   ```

3. **Save the PM2 process**
   ```bash
   pm2 save
  ```

   ```
##  6️⃣ Nginx Reverse Proxy Setup

1. **Install NGINX**
   ```bash
   sudo apt install nginx -y
2. **Created config file for my domain**
   ```bash
    sudo nano /etc/nginx/sites-available/standcore.mooo.com
   ```

3. **Enabled the config**
   ```bash
   sudo nano /etc/nginx/sites-available/yourdomain.com
   ```

   **Example config**
  ```
server {
    listen 80;
    server_name standcore.mooo.com www.standcore.mooo.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl;
    server_name standcore.mooo.com www.standcore.mooo.com;

    ssl_certificate /etc/letsencrypt/live/standcore.mooo.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/standcore.mooo.com/privkey.pem;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}


   ```

4. **Enable the Config**
   ```bash
   sudo ln -s /etc/nginx/sites-available/yourdomain.com /etc/nginx/sites-enabled/
   ```

5. **Test the NGINX Config**
   ```bash
   sudo nginx -t
   ```

6. **Restart NGINX**
   ```bash
   sudo systemctl restart nginx
   ```

---

##  7️⃣ Secure with HTTPS (Let’s Encrypt)

1. **Install Certbot**
   ```bash
   sudo apt install certbot python3-certbot-nginx -y
   ```

2. **Obtain & Install SSL**
   ```bash
   sudo certbot --nginx -d standcore.mooo.com
   ```

3. **Allowed HTTPS through firewall**
   ```bash
   sudo ufw allow 443/tcp
   ```

---

## GitHub Deployment!
**Initialized Git**
```
git init
git remote add origin git@github.com:kvvngdee/Standcore.git
```
**Pushed code**
```
git add .
git commit -m "Initial commit"
git push -u origin main
```

- Your Node.js app is running.
- NGINX is serving it securely via HTTPS.
- PM2 keeps it alive.

---
## Verification to confirm everything works well!

**Checked site with:**
```bash
curl -I https://standcore.mooo.com
```
**Expected response:**
```
HTTP/1.1 200 OK
Server: nginx
X-Powered-By: Express
```
## Page is hosted at
*IP address http://108.129.253.153/
*URL https://standcore.mooo.com/

## Common Commands

| Purpose                | Command                                  |
|------------------------|------------------------------------------|
| Check NGINX status     | `sudo systemctl status nginx`            |
| Restart NGINX          | `sudo systemctl restart nginx`           |
| Restart your app with PM2 | `pm2 restart all`                    |
| View app logs          | `pm2 logs`                               |
| View PM2 process list  | `pm2 list`                               |

## Screenshot of the domain unsecured
![Screenshot from 2025-06-13 19-34-03](https://github.com/user-attachments/assets/ffaecbc3-ea1b-4137-bb77-dab8e1b74abf)
## Screenshot of the domain secured
![Screenshot from 2025-06-15 05-41-11](https://github.com/user-attachments/assets/647236ce-c112-45ff-97b4-713a2c45c2f4)


