# Nginx Reverse Proxy Setup

Nginx is used as a reverse proxy to protect internal services. Instead of exposing ports directly to the internet, all traffic goes through Nginx on port 80. This means services like n8n and Grafana are not reachable by their internal ports from outside the server.

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

        # Grafana
        location /grafana/ {
            proxy_pass http://grafana:3000/grafana/;
            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
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

## 4. Connect existing containers to the network

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

## 5. Make sure databases have no public ports

Recreate MySQL and PostgreSQL without exposing their ports:

```bash
docker rm -f mysql-container
docker run -d \
  --name mysql-container \
  --network nginx-network \
  -e MYSQL_ROOT_PASSWORD=YOUR_PASSWORD \
  -v mysql_data:/var/lib/mysql \
  mysql

docker rm -f instance-prostgress
docker run -d \
  --name instance-prostgress \
  --network nginx-network \
  -e POSTGRES_PASSWORD=YOUR_PASSWORD \
  -e PGDATA=/var/lib/postgresql/18/docker \
  -v postgres_data:/var/lib/postgresql \
  postgres
```

## 6. Verify the setup

```bash
docker ps --format "table {{.Names}}\t{{.Ports}}"
```

Only Nginx should show `0.0.0.0:80` and `0.0.0.0:443`. All other containers should have no public binding.

## 7. Test the proxy

```bash
curl -I http://YOUR_SERVER_IP
curl -I http://YOUR_SERVER_IP/grafana/
```

You should get a `200 OK` for n8n and a `302` redirect to `/grafana/login` for Grafana — both mean the proxy is working correctly.