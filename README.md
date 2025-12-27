# MERN Stack Deployment on VPS Server (Ubuntu)

This document provides a **clean, standard, step-by-step guide** to deploy a **MERN application (Next.js frontend + Express backend + MongoDB)** on an **Ubuntu EC2 instance** using **PM2, Nginx, SSL (Let's Encrypt)**, and **Swap memory**.

- Preparing the VPS Environment
- Setting Up the MongoDB Database
- Deploying the Express and Node.js Backend
- Deploying the React Frontends
- Configuring Nginx as a Reverse Proxy
- Setting Up SSL Certificates
---

## 1️⃣ Connect to EC2 Instance

```bash
ssh -i your-key.pem ubuntu@<EC2_PUBLIC_IP>
```

Update system packages:

```bash
sudo apt update && sudo apt upgrade -y
```

---

## 2️⃣ Install Node.js 20 LTS & Git

```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs git
```

Verify installation:

```bash
node -v
npm -v
```

Install PM2 globally:

```bash
sudo npm install -g pm2
```

---

## 3️⃣ Install & Configure Nginx

```bash
sudo apt install -y nginx
sudo systemctl enable nginx
sudo systemctl start nginx
```

Verify:

```bash
curl http://localhost
```

---

## 4️⃣ Recommended Project Structure

```
/var/www/mern-app
 ├── backend
 │    ├── server.js
 │    ├── package.json
 │    └── .env
 └── frontend
      ├── build / .next
      └── package.json
```

Create directory and set permissions:

```bash
sudo mkdir -p /var/www/mern-app
sudo chown -R ubuntu:ubuntu /var/www/mern-app
cd /var/www/mern-app
```

---

## 5️⃣ Firewall Configuration (UFW)

Check firewall status:

```bash
sudo ufw status
```

Enable and allow required ports:

```bash
sudo ufw enable
sudo ufw default deny incoming
sudo ufw default allow outgoing

sudo ufw allow OpenSSH
sudo ufw allow 'Nginx Full'

sudo ufw enable
sudo ufw status
```

---

## 6️⃣ Install & Configure MongoDB (Local VPS)

### Install MongoDB 8.0

```bash
sudo apt-get install gnupg curl
curl -fsSL https://www.mongodb.org/static/pgp/server-8.0.asc | \
 sudo gpg -o /usr/share/keyrings/mongodb-server-8.0.gpg --dearmor
```

Create MongoDB list file:

```bash
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/8.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list
```

Install MongoDB:

```bash
sudo apt-get update
sudo apt-get install -y mongodb-org
```

Start & enable MongoDB:

```bash
sudo systemctl daemon-reload
sudo systemctl start mongod
sudo systemctl enable mongod
```

Verify status:

```bash
sudo systemctl status mongod
```

MongoDB Connection URI:

```
mongodb://127.0.0.1:27017
```

Allow MongoDB port (optional if local only):

```bash
sudo ufw allow 27017
sudo service mongod restart
```

---

## 7️⃣ Run Backend with PM2

```bash
cd /var/www/mern-app/backend
pm2 start server.js --name "mern-backend"
pm2 save
```

Verify:

```bash
pm2 list
```

---

## 8️⃣ Run Next.js Frontend with PM2 (IMPORTANT)

```bash
cd /var/www/mern-app/frontend
pm2 start npm --name "next-frontend" -- start
pm2 save
```

Verify:

```bash
pm2 list
```

---

## 9️⃣ Nginx Configuration (Next.js + Express API)

Create Nginx config:

```bash
sudo nano /etc/nginx/sites-available/your-domain.com
```

### ✅ Production Nginx Configuration

```nginx
server {
    listen 80;
    server_name your-domain.com www.your-domain.com;

    # Frontend (Next.js)
    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    # Backend API
    location /api/ {
        proxy_pass http://localhost:5000/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

Enable config & reload:

```bash
sudo rm /etc/nginx/sites-enabled/default
sudo ln -s /etc/nginx/sites-available/your-domain.com /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

## 🔐 10️⃣ Enable HTTPS (SSL) with Let’s Encrypt

Install Certbot:

```bash
sudo apt install -y certbot python3-certbot-nginx
```

Generate SSL:

```bash
sudo certbot --nginx -d your-domain.com -d www.your-domain.com
```

Auto-renew test:

```bash
sudo certbot renew --dry-run
```

---

## 11️⃣ Add Swap Memory (Highly Recommended)

### Create 2GB Swap (Best for Low RAM EC2)

```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

Make swap permanent:

```bash
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

Verify:

```bash
free -h
```

Expected Output:

```
Swap: 2.0G
```

---

## ✅ Final Checklist

* ✔ Node.js 20 installed
* ✔ MongoDB running
* ✔ PM2 managing frontend & backend
* ✔ Nginx reverse proxy configured
* ✔ SSL enabled
* ✔ Firewall secured
* ✔ Swap added (no build crash)

---

🎉 **Your MERN application is now production-ready on AWS EC2.**

If you still need help in deployment:

Contact us on email : harikrishnanr.dev@gmail.com
