# 🚀 Deploy Next.js Portfolio to VPS
> Access your portfolio at `http://your_server_ip` using GitHub, PM2, and Nginx.

---

## What We'll Use

| Tool | Purpose |
|------|---------|
| **Git** | Clone your repo from GitHub |
| **Node.js + Yarn** | Build your Next.js app |
| **PM2** | Keep your app running 24/7 |
| **Nginx** | Serve traffic on port 80 (HTTP) |

---

## Step 1 — Update Your Server

```bash
apt update && apt upgrade -y
```

---

## Step 2 — Install Node.js (v20 LTS)

```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | bash -
apt install -y nodejs
```

Verify:

```bash
node --version
npm --version
```

---

## Step 3 — Install Yarn

```bash
npm install -g yarn
```

---

## Step 4 — Install PM2

PM2 keeps your app alive after crashes and server reboots.

```bash
npm install -g pm2
```

---

## Step 5 — Install Nginx

```bash
apt install -y nginx
```

Start and enable Nginx:

```bash
systemctl start nginx
systemctl enable nginx
```

---

## Step 6 — Clone Your GitHub Repo

### Option A — Public Repo (Easiest, no keys needed)

```bash
cd /var/www
git clone https://github.com/syed-taimoor/Syed-Taimoor-Portfolio.git
cd Syed-Taimoor-Portfolio
```

### Option B — Private Repo (SSH Key Setup required)

**6.1 — Generate an SSH key on your server:**

```bash
ssh-keygen -t ed25519 -C "taimoor-ai-labs-server"
```

Press `Enter` three times (no passphrase needed).

**6.2 — Copy the public key:**

```bash
cat ~/.ssh/id_ed25519.pub
```

Copy the entire output line that appears.

**6.3 — Add the key to GitHub:**

1. Go to **github.com** → avatar (top right) → **Settings**
2. Left sidebar → **SSH and GPG keys**
3. Click **New SSH key**
4. Title: `Taimoor-AI-Labs Server`
5. Paste your key → **Add SSH key**

**6.4 — Test the connection:**

```bash
ssh -T git@github.com
```

Expected response:
```
Hi syed-taimoor! You've successfully authenticated...
```

**6.5 — Clone using SSH:**

```bash
cd /var/www
git clone git@github.com:syed-taimoor/Syed-Taimoor-Portfolio.git
cd Syed-Taimoor-Portfolio
```

---

## Step 7 — Set Up Environment Variables

```bash
cp .env.example .env
nano .env
```

Fill in your values:

```env
HOST=your_server_ip
POSTGRES_USER=admin
POSTGRES_PASSWORD=your_strong_password
POSTGRES_DB=mydb
```

Save and exit: `Ctrl + X` → `Y` → `Enter`

---

## Step 8 — Install Dependencies & Build

```bash
yarn install
yarn build
```

> This creates the `.next` production build folder.

---

## Step 9 — Start App with PM2

```bash
pm2 start yarn --name "portfolio" -- start
```

Save PM2 process list so it restarts on reboot:

```bash
pm2 save
pm2 startup
```

Run the command that `pm2 startup` outputs (it looks like):

```bash
env PATH=$PATH:/usr/bin pm2 startup systemd -u root --hp /root
```

### Useful PM2 Commands

```bash
pm2 status          # Check if app is running
pm2 logs portfolio  # View live logs
pm2 restart portfolio
pm2 stop portfolio
```

---

## Step 10 — Configure Nginx

Remove the default config and create yours:

```bash
rm /etc/nginx/sites-enabled/default
nano /etc/nginx/sites-available/portfolio
```

Paste this config:

```nginx
server {
    listen 80;
    server_name your_server_ip;

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

> Replace `your_server_ip` with your actual server IP.

Save and exit: `Ctrl + X` → `Y` → `Enter`

Enable the config:

```bash
ln -s /etc/nginx/sites-available/portfolio /etc/nginx/sites-enabled/
```

Test Nginx config:

```bash
nginx -t
```

Reload Nginx:

```bash
systemctl reload nginx
```

---

## Step 11 — Open Firewall Port

Allow HTTP traffic:

```bash
ufw allow 80
ufw allow 22
ufw enable
```

---

## ✅ Done! Test It

Open your browser and visit:

```
http://your_server_ip
```

Your portfolio should load! 🎉

---

## How to Update Your Site (After Git Push)

Whenever you push new changes to GitHub, SSH into your server and run:

```bash
cd /var/www/Syed-Taimoor-Portfolio
git pull
yarn install
yarn build
pm2 restart portfolio
```

---

## Troubleshooting

| Issue | Fix |
|-------|-----|
| Site not loading | Check `pm2 status` and `pm2 logs portfolio` |
| Nginx error | Run `nginx -t` to find config errors |
| Port 3000 not responding | Run `pm2 restart portfolio` |
| Changes not showing | Run `yarn build` then `pm2 restart portfolio` |
| Permission denied (server) | Make sure you're running as `root` |
| `git@github.com: Permission denied (publickey)` | Follow Step 6 Option B — SSH key not added to GitHub |

---

## Full Architecture

```
Browser (http://your_server_ip)
        │
        ▼
    Nginx :80
        │  proxy_pass
        ▼
  Next.js App :3000
   (managed by PM2)
        │
        ▼
  PostgreSQL :5432
   (Docker container)
```

---

> **Tip:** Want HTTPS with a domain? Point your domain to this IP, then run:
> ```bash
> apt install -y certbot python3-certbot-nginx
> certbot --nginx -d yourdomain.com
> ```
