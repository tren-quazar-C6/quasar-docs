# Nginx Reverse Proxy Setup

Nginx is used as a reverse proxy to protect internal services. Instead of exposing ports directly to the internet, all web traffic goes through Nginx on port 80. This means services like n8n and Grafana are not reachable by their internal ports from outside the server.

## What is a Reverse Proxy?

A reverse proxy sits in front of your services and forwards requests to them internally. The outside world only sees Nginx — the internal ports are never exposed.

## 1. Create the Docker network

All containers must share the same network so they can communicate internally.

```bash
docker network create nginx-network
```

## 2. Create the Nginx configuration file

```bash
mkdir -p ~/nginx

cat > ~/nginx/nginx.conf << 'EOF'
events {}
http {
    server {
        listen 80;
        server_name YOUR_SERVER_IP;

        # Grafana (must be declared before / to take priority)
        location /grafana/ {
            proxy_pass http://grafana:3000/grafana/;
            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        # n8n
        location / {
            proxy_pass http://n8n:5678;
            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_read_timeout 300s;
        }
    }
}
EOF
```

## 3. Run the Nginx container

```bash
docker run -d \
  --name nginx \
  --network nginx-network \
  -p 80:80 \
  -p 443:443 \
  -v ~/nginx/nginx.conf:/etc/nginx/nginx.conf \
  nginx:alpine
```

## 4. Connect all containers to the network

```bash
docker network connect nginx-network n8n
docker network connect nginx-network grafana
docker network connect nginx-network prometheus
docker network connect nginx-network mysql-container
docker network connect nginx-network instance-prostgress
docker network connect nginx-network node-exporter
docker network connect nginx-network mysql-exporter
docker network connect nginx-network postgres-exporter
```

## 5. Database ports

Databases (MySQL and PostgreSQL) are kept with their ports open for development purposes, since the team needs to connect to them directly from local machines. The passwords are strong enough to protect them without closing the ports.

```bash
# MySQL with port open
docker run -d \
  --name mysql-container \
  --network nginx-network \
  -p 3306:3306 \
  -e MYSQL_ROOT_PASSWORD=YOUR_PASSWORD \
  -v mysql_data:/var/lib/mysql \
  mysql

# PostgreSQL with port open
docker run -d \
  --name instance-prostgress \
  --network nginx-network \
  -p 5432:5432 \
  -e POSTGRES_PASSWORD=YOUR_PASSWORD \
  -e PGDATA=/var/lib/postgresql/18/docker \
  -v postgres_data:/var/lib/postgresql \
  postgres
```

## 6. Monitoring stack setup

The monitoring stack (Prometheus, Grafana, and exporters) runs fully internal — none of their ports are exposed publicly.

### MySQL Exporter

The newer version of `mysqld-exporter` requires credentials in a `.my.cnf` file instead of environment variables:

```bash
mkdir -p /etc/mysql-exporter

cat > /etc/mysql-exporter/.my.cnf << 'EOF'
[client]
user=root
password=YOUR_MYSQL_PASSWORD
host=mysql-container
port=3306
EOF

docker run -d \
  --name mysql-exporter \
  --network nginx-network \
  -v /etc/mysql-exporter/.my.cnf:/.my.cnf \
  prom/mysqld-exporter:latest \
  --mysqld.address=mysql-container:3306 \
  --mysqld.username=root \
  --config.my-cnf=/.my.cnf
```

### Prometheus

Mount your existing config file so scrape targets survive container restarts:

```bash
docker run -d \
  --name prometheus \
  --network nginx-network \
  -v /root/monitoring/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus:latest
```

Your `prometheus.yml` should include all exporters:

```yaml
global:
  scrape_interval: 5s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['prometheus:9090']

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']

  - job_name: 'mysql'
    static_configs:
      - targets: ['mysql-exporter:9104']

  - job_name: 'postgres'
    static_configs:
      - targets: ['postgres-exporter:9187']
```

### Grafana

Run Grafana with a persistent volume and subpath configuration so dashboards survive restarts and it works correctly behind Nginx:

```bash
docker run -d \
  --name grafana \
  --network nginx-network \
  -e GF_SERVER_ROOT_URL=http://YOUR_SERVER_IP/grafana \
  -e GF_SERVER_SERVE_FROM_SUB_PATH=true \
  -v grafana_data:/var/lib/grafana \
  grafana/grafana:latest
```

### Grafana dashboard setup

After logging in with `admin` / `admin`:

1. Go to **Connections → Data Sources → Add → Prometheus**
2. Set the URL to `http://prometheus:9090`
3. Click **Save & Test**

To import the MySQL dashboard:
1. Go to **Dashboards → Import**
2. Enter ID `7362` → click **Load**
3. Select your Prometheus datasource → **Import**

If the dashboard shows "Renamed or missing variables", go to **⚙️ Settings → Variables → New variable** and add:
- **Type:** Datasource
- **Name:** `DS_PROMETHEUS`
- **Plugin:** Prometheus

## 7. Verify the setup

```bash
docker ps --format "table {{.Names}}\t{{.Ports}}"
```

Only Nginx should show `0.0.0.0:80` and `0.0.0.0:443`. The monitoring stack should show no public bindings.

## 8. Test the proxy

```bash
curl -I http://YOUR_SERVER_IP
curl -I http://YOUR_SERVER_IP/grafana/
```

You should get a `200 OK` for n8n and a `302` redirect to `/grafana/login` for Grafana — both mean the proxy is working correctly.