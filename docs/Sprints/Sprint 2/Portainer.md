# Portainer Setup

Portainer is a web-based UI for managing Docker containers. Instead of using the terminal to check logs, restart containers, or inspect volumes, Portainer lets you do all of that from your browser.

## 1. Run the Portainer container

```bash
docker run -d \
  --name portainer \
  --network nginx-network \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:latest
```

:::note
Do **not** add `-p 9000:9000`. Nginx handles all traffic — Portainer stays internal.
:::

## 2. Add Portainer to the Nginx config

Edit `~/nginx/nginx.conf` and add the Portainer location block before the `location /` block:

```nginx
# Portainer
location /portainer/ {
    proxy_pass http://portainer:9000/;
    proxy_http_version 1.1;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
}
```

## 3. Reload Nginx

```bash
docker exec nginx nginx -s reload
```

## 4. First time setup

Open your browser and go to:

```
http://YOUR_SERVER_IP/portainer/
```

:::warning
Portainer has a short timeout on first access. Create your admin account immediately after opening it, otherwise it will lock and you'll need to restart the container.
:::

1. Enter a username and a strong password
2. Click **Create user**
3. On the next screen click **Get Started** to connect to the local Docker environment

## 5. What you can do in Portainer

Once inside, click on **local** to access your Docker environment. The most useful sections are:

**Containers** — see all running containers at a glance. From here you can start, stop, restart, view logs, and open a terminal inside any container without using SSH.

**Volumes** — inspect persistent data volumes like `grafana_data`, `portainer_data`, and your database volumes.

**Networks** — verify that all containers are correctly connected to `nginx-network`.

**Images** — see all Docker images downloaded on the server.

## 6. Verify all containers are running

After logging in, go to **Containers** and confirm all of these show as **Running** (green dot):

- nginx
- n8n
- grafana
- prometheus
- mysql-container
- instance-prostgress
- node-exporter
- mysql-exporter
- postgres-exporter
- portainer