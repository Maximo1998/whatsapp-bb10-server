# WhatsApp Server for BlackBerry 10

A Node.js server that bridges the WhatsApp network with the [whatsapp-bb10-client](https://github.com/Maximo1998/whatsapp-bb10-client) Android app running on BlackBerry 10.

[![Buy Me A Coffee](https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png)](https://buymeacoffee.com/maxlakh1m)

## Requirements

- Node.js 16+
- A server with a public HTTPS URL (e.g. via nginx + Let's Encrypt)
- A WhatsApp account to link

## Installation

```bash
git clone https://github.com/Maximo1998/whatsapp-bb10-server.git
cd whatsapp-bb10-server
npm install
```

## Running

```bash
node server.js
```

For production use with auto-restart:

```bash
npm install -g pm2
pm2 start server.js --name whatsapp-bb10-server
pm2 save
```

The server listens on port **3000** by default. Put it behind a reverse proxy (nginx/Caddy) with HTTPS — the BB10 client requires HTTPS.

---

## Making the server accessible from anywhere

The BB10 client needs to reach the server over the internet with a valid HTTPS URL. Here is the recommended setup.

### 1. Get a VPS (cloud server)

A cheap VPS (Virtual Private Server) gives you a public IP and lets the server run 24/7. Good options:

| Provider | Starting price | Notes |
|----------|---------------|-------|
| [Hetzner](https://www.hetzner.com/cloud) | ~€4/month | Europe, very good value |
| [DigitalOcean](https://www.digitalocean.com) | ~$6/month | Beginner-friendly |
| [Vultr](https://www.vultr.com) | ~$6/month | Many locations |
| [Oracle Cloud](https://www.oracle.com/cloud/free/) | Free tier | ARM instance, always free |

Any Ubuntu 22.04 or Debian 12 instance works fine.

### 2. Get a domain name

A domain lets you use `https://your-name.com` instead of a raw IP. Registrars:

- [Namecheap](https://www.namecheap.com) — cheap first year (~$10/year)
- [Cloudflare Registrar](https://www.cloudflare.com/products/registrar/) — at-cost pricing, no markup
- [Porkbun](https://porkbun.com) — often has $1–2 first-year deals

### 3. Point the domain to your server

In your domain's DNS panel, add an **A record**:

| Type | Name | Value | TTL |
|------|------|-------|-----|
| A | `@` (or `whatsapp`) | `YOUR_SERVER_IP` | Auto |

Wait a few minutes for DNS to propagate.

### 4. Get a free HTTPS certificate (Let's Encrypt)

On your VPS, install Certbot and nginx:

```bash
sudo apt update
sudo apt install nginx certbot python3-certbot-nginx -y

# Replace your-domain.com with your actual domain
sudo certbot --nginx -d your-domain.com
```

Certbot configures nginx and renews the certificate automatically.

### 5. Configure nginx as a reverse proxy

Edit `/etc/nginx/sites-available/default` (or create a new site file):

```nginx
server {
    listen 443 ssl;
    server_name your-domain.com;

    ssl_certificate     /etc/letsencrypt/live/your-domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your-domain.com/privkey.pem;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}

server {
    listen 80;
    server_name your-domain.com;
    return 301 https://$host$request_uri;
}
```

```bash
sudo nginx -t && sudo systemctl reload nginx
```

### 6. Open the firewall

```bash
sudo ufw allow 80
sudo ufw allow 443
sudo ufw allow 22   # keep SSH open!
sudo ufw enable
```

Your server is now reachable at `https://your-domain.com` from any network.

> **Home server alternative:** If you want to run this on a home PC instead of a VPS, you will need to configure port forwarding on your router (ports 80 and 443 → your PC's local IP) and use a dynamic DNS service like [DuckDNS](https://www.duckdns.org) (free) if your ISP assigns a dynamic IP.

---

## First-time WhatsApp Login

1. Start the server
2. Open `https://your-domain.com/login` in a browser
3. Enter your phone number and country
4. Scan the QR code with the WhatsApp app on your phone
5. Once authenticated, open the BB10 client app and connect

## OTA Updates

The server serves `version.json` so the BB10 client can check for app updates automatically:

```json
{
  "version": "1.5.2",
  "apk_url": "https://github.com/Maximo1998/whatsapp-bb10-client/releases/download/v1.5.2/app-debug.apk"
}
```

Edit this file whenever you publish a new APK release.

## API Reference

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/startclient/:mobile` | GET | Start a WhatsApp session |
| `/login-status/:mobile` | GET | Poll authentication status |
| `/api/login/:mobile` | GET | Get user info after login |
| `/api/chats/:mobile` | GET | List chats |
| `/api/messages/:mobile/:chatId` | GET | Get messages for a chat |
| `/api/sendmessage` | POST | Send a text message |
| `/api/mediafile/:filename` | GET | Serve media (images, videos, stickers) |
| `/api/version` | GET | Return current version info for OTA |
| `/login` | GET | Web UI for QR login |

## Credits

Based on [NovelProfessor's whatsapp-client-android](https://github.com/NovelProfessor/whatsapp-client-android).
