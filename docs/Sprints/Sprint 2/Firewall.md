# Firewall Setup with UFW

UFW (Uncomplicated Firewall) is the firewall tool used on the server to control which ports are accessible from the internet. It adds an extra layer of protection on top of Nginx — even if a container accidentally exposes a port, UFW will block it at the network level.

## How it works

UFW works by defining rules for incoming and outgoing traffic. The default policy blocks all incoming connections, and only the ports explicitly allowed can receive traffic from the internet.

## 1. Check current status

```bash
ufw status verbose
```

## 2. Set default rules

```bash
# Block all incoming connections by default
ufw default deny incoming

# Allow all outgoing connections
ufw default allow outgoing
```

## 3. Allow only the required ports

```bash
# SSH — always do this first to avoid locking yourself out
ufw allow 22/tcp

# HTTP and HTTPS — required for Nginx
ufw allow 80/tcp
ufw allow 443/tcp

# MySQL — open for local development access
ufw allow 3306/tcp

# PostgreSQL — open for local development access
ufw allow 5432/tcp
```

:::warning
Always allow port 22 **before** enabling UFW. If you enable the firewall without allowing SSH, you will lose access to your server.
:::

## 4. Enable the firewall

```bash
ufw enable
```

Type `y` when prompted to confirm.

## 5. Verify the configuration

```bash
ufw status verbose
```

Expected output:

```
Status: active
Default: deny (incoming), allow (outgoing)

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere
80/tcp                     ALLOW IN    Anywhere
443/tcp                    ALLOW IN    Anywhere
3306/tcp                   ALLOW IN    Anywhere
5432/tcp                   ALLOW IN    Anywhere
```

## What is protected

With this configuration, all internal service ports are blocked from the internet at the firewall level, even if a container exposes them:

| Port | Service | Internet access |
|------|---------|----------------|
| 22 | SSH | ✅ Open |
| 80 | Nginx HTTP | ✅ Open |
| 443 | Nginx HTTPS | ✅ Open |
| 3306 | MySQL | ✅ Open (dev only) |
| 5432 | PostgreSQL | ✅ Open (dev only) |
| 3000 | Grafana | 🔒 Blocked (via Nginx) |
| 5678 | n8n | 🔒 Blocked (via Nginx) |
| 9000 | Portainer | 🔒 Blocked (via Nginx) |
| 9090 | Prometheus | 🔒 Blocked |
| 9100 | Node Exporter | 🔒 Blocked |
| 9104 | MySQL Exporter | 🔒 Blocked |
| 9187 | Postgres Exporter | 🔒 Blocked |