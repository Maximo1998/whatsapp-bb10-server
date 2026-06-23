# WhatsApp Server for BlackBerry 10

A Node.js server that bridges the WhatsApp network with the [whatsapp-bb10-client](https://github.com/Maximo1998/whatsapp-bb10-client) Android app running on BlackBerry 10.

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

### Example nginx config

```nginx
server {
    listen 443 ssl;
    server_name your-server.com;

    ssl_certificate     /etc/letsencrypt/live/your-server.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your-server.com/privkey.pem;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
    }
}
```

## First-time WhatsApp Login

1. Start the server
2. Open `https://your-server.com/login` in a browser
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
