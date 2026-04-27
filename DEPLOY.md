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
docker compose exec marzban marzban-cli admin create --sudo
```

The dashboard is available at:

```text
http://SERVER_IP:8000/dashboard/
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
