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

`ALLOW_INSECURE_HTTP = True` is enabled so Marzban can bind to `0.0.0.0:8000` behind Nginx Proxy Manager. Do not expose port `8000` publicly unless it is limited to your trusted network.

## Run

```bash
docker compose up -d --build
docker compose logs -f
```

Create the first sudo admin:

```bash
docker compose exec marzban marzban-cli admin create -u admin --sudo
```

Marzban runs on `0.0.0.0:8000` because `ALLOW_INSECURE_HTTP = True` is enabled. Put it behind Nginx Proxy Manager and let NPM handle TLS for the public domain.

## Nginx Proxy Manager

Create a Proxy Host for the public domain:

```text
Domain Names: firezone.mark-sandbox.ru
Scheme: http
Forward Hostname / IP: SERVER_IP
Forward Port: 8000
Websockets Support: enabled
Block Common Exploits: enabled
SSL: request a new Let's Encrypt certificate
Force SSL: enabled
```

For internal access without a certificate, use:

```text
http://SERVER_INTERNAL_IP:8000/dashboard/
```

The dashboard will be available at:

```text
https://firezone.mark-sandbox.ru/dashboard/
http://SERVER_INTERNAL_IP:8000/dashboard/
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
