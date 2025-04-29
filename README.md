This repository contains examples of setting up self-steal configurations using Xray and sing-box in Docker.

Both setups use a fake Confluence login page, but you can use any page you like in `caddy/templates/index.html`

Based on Akiyamov [xray-vps-setup](https://github.com/Akiyamov/xray-vps-setup)

> [!WARNING]
> Domain is required to use this setup

- [Xray](#xray)
- [sing-box](#sing-box)

## Prerequisites

Install git:

```bash
sudo apt install git
```

Install Docker:

```bash
bash <(wget -qO- https://get.docker.com)
```

If you're using non-root account and you want to run Docker without sudo, add your user to the docker group:

```bash
sudo groupadd docker
sudo usermod -aG docker $USER
```

> [!IMPORTANT]
> Log out and log back in to apply the changes

Clone the repository:

```bash
git clone https://github.com/vernette/selfsteal-examples
cd selfsteal-examples
```

Change `$VLESS_DOMAIN` to your domain name in `caddy/Caddyfile`. For example, `testdomain.com`:

```bash
sed -i 's/\$VLESS_DOMAIN/testdomain.com/g' caddy/Caddyfile
```

## Xray

Copy compose file:

```bash
cp docker-compose-xray.yml docker-compose.yml
```

Generate required values

```bash
# Generate private and public keys ($PRIVATE_KEY and $PUBLIC_KEY)
docker run --rm ghcr.io/xtls/xray-core:25.1.1 x25519

# Generate UUID ($UUID)
docker run --rm ghcr.io/xtls/xray-core:25.1.1 uuid

# Generate SID ($SHORT_ID)
openssl rand -hex 8
```

Replace `$UUID`, `$PRIVATE_KEY`, `$SHORT_ID` and `$VLESS_DOMAIN` in `xray/config.json` with generated values:

```json
"inbounds": [
  {
    "tag": "VLESS TCP VISION REALITY",
    "protocol": "vless",
    "listen": "0.0.0.0",
    "port": 443,
    "settings": {
      "clients": [
        {
          "email": "user",
          "id": "$UUID",
          "flow": "xtls-rprx-vision"
        }
      ],
      "decryption": "none"
    },
    "streamSettings": {
      "network": "tcp",
      "security": "reality",
      "realitySettings": {
        "xver": 1,
        "dest": "127.0.0.1:4123",
        "serverNames": ["$VLESS_DOMAIN"],
        "privateKey": "$PRIVATE_KEY",
        "shortIds": ["$SHORT_ID"]
      }
    },
    "sniffing": {
      "enabled": true,
      "destOverride": ["http", "tls"],
      "routeOnly": true
    }
  }
]
```

Start services:

```bash
docker compose up -d
```

## sing-box

Copy compose file:

```bash
cp docker-compose-singbox.yml docker-compose.yml
```

Generate required values

```bash
# Generate private and public keys ($PRIVATE_KEY and $PUBLIC_KEY)
docker run --rm ghcr.io/sagernet/sing-box:v1.11.8 generate reality-keypair

# Generate UUID ($UUID)
docker run --rm ghcr.io/sagernet/sing-box:v1.11.8 generate uuid

# Generate SID ($SHORT_ID)
openssl rand -hex 8
```

Replace `$UUID`, `$PRIVATE_KEY`, `$SHORT_ID` and `$VLESS_DOMAIN` in `sing-box/config.json` with generated values:

```json
"inbounds": [
  {
    "tag": "VLESS TCP VISION REALITY",
    "type": "vless",
    "listen": "0.0.0.0",
    "listen_port": 443,
    "users": [
      {
        "name": "user",
        "uuid": "$UUID",
        "flow": "xtls-rprx-vision"
      }
    ],
    "tls": {
      "enabled": true,
      "server_name": "$VLESS_DOMAIN",
      "reality": {
        "enabled": true,
        "handshake": {
          "server": "127.0.0.1",
          "server_port": 4123
        },
        "private_key": "$PRIVATE_KEY",
        "short_id": ["$SHORT_ID"]
      }
    }
  }
]
```

Start services:

```bash
docker compose up -d
```

## VLESS URL template

`vless://$UUID@$VLESS_DOMAIN:443?security=reality&sni=$VLESS_DOMAIN&fp=chrome&pbk=$PUBLIC_KEY&sid=$SHORT_ID&spx=/&type=tcp&flow=xtls-rprx-vision&encryption=none#selfsteal-test`
