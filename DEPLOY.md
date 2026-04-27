# Deploy

This project is configured to run Marzban from the local source tree with MariaDB.

## Server setup

Install Docker and Docker Compose plugin on the server, then clone this repository:

```bash
cd /opt
git clone <your-repository-url> marzban
cd /opt/marzban
```

Create the runtime environment file:

```bash
cp .env.example .env
nano .env
```

Before starting, change these values in `.env`:

```env
MARIADB_PASSWORD=change-this-password
MARIADB_ROOT_PASSWORD=change-this-root-password
SQLALCHEMY_DATABASE_URL = "mysql+pymysql://marzban:change-this-password@127.0.0.1:3306/marzban"
```

The password in `SQLALCHEMY_DATABASE_URL` must match `MARIADB_PASSWORD`.

## Run

```bash
docker compose up -d --build
docker compose logs -f
```

Create the first sudo admin:

```bash
docker compose exec marzban marzban-cli admin create -u admin --sudo
```

Marzban runs on `127.0.0.1:8000` when SSL files are not configured. Use Nginx to expose it through the public domain and the internal IP.

## Nginx

Install Nginx and Certbot:

```bash
sudo apt update
sudo apt install -y nginx certbot python3-certbot-nginx
```

Create `/etc/nginx/sites-available/marzban`:

```nginx
server {
    listen 80;
    server_name firezone.mark-sandbox.ru INTERNAL_SERVER_IP;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Replace `INTERNAL_SERVER_IP` with the real internal server IP. Enable the site:

```bash
sudo ln -s /etc/nginx/sites-available/marzban /etc/nginx/sites-enabled/marzban
sudo nginx -t
sudo systemctl reload nginx
```

Issue the public certificate:

```bash
sudo certbot --nginx -d firezone.mark-sandbox.ru
```

The dashboard will be available at:

```text
https://firezone.mark-sandbox.ru/dashboard/
http://INTERNAL_SERVER_IP/dashboard/
```

## Proxy Ports

The default `xray_config.json` enables these inbound ports:

```text
1080 - Shadowsocks TCP/UDP
2082 - VMess WebSocket, path /vmess
2083 - VLESS TCP
2084 - VLESS WebSocket, path /vless
2085 - VLESS gRPC, service vless-grpc
2086 - VLESS HTTPUpgrade, path /vless-upgrade
2087 - VLESS SplitHTTP, path /vless-splithttp
2096 - Trojan TCP
2097 - VMess gRPC, service vmess-grpc
2098 - Trojan WebSocket, path /trojan
2099 - Trojan gRPC, service trojan-grpc
2100 - VLESS mKCP
2101 - VMess mKCP
```

Open these ports on the server firewall if you want to use all protocols.
Port `443` is intentionally left free for Nginx/Certbot and `firezone.mark-sandbox.ru`.

For production, use a domain with TLS and open only the required ports.
