## VPS Monitoring with Prometheus, Grafana & Node Exporter

This guide covers how to set up a full monitoring stack on a Linux VPS using Docker Compose. By the end, you will have Prometheus collecting metrics, Node Exporter exposing host-level system data, and Grafana displaying everything in a dashboard.

## Architecture overview

```
Node Exporter (:9100) ──scrapes──> Prometheus (:9090) ──queries──> Grafana (:3000)
```

- **Node Exporter** reads host metrics directly from the Linux kernel (`/proc`, `/sys`).
- **Prometheus** scrapes those metrics on a fixed interval and stores them.
- **Grafana** connects to Prometheus as a data source and renders dashboards.

---

## Prerequisites

- A Linux VPS with Docker and Docker Compose installed.
- SSH access to the server (this guide uses [Termius](https://termius.com)).
- Basic familiarity with the terminal.

---

## Project structure

Create the following folder structure on your VPS under `/root/monitoring`:

```
/root/monitoring/
├── docker-compose.yml
└── prometheus/
    └── prometheus.yml
```

---

## Step 1 — Configure Prometheus

Create `prometheus/prometheus.yml` with a scrape job for each target:

```yaml
global:
  scrape_interval: 5s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['prometheus:9090']

  - job_name: 'docker'
    static_configs:
      - targets: ['host.docker.internal:9323']

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']
```

> **Note:** Inside Docker, containers communicate by service name, not `localhost`. That is why the targets use names like `prometheus` and `node-exporter` instead of `127.0.0.1`.

---

## Step 2 — Configure Docker Compose

Create `docker-compose.yml` with three services — Prometheus, Grafana, and Node Exporter — all sharing the same `monitoring` network:

```yaml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    restart: always
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
    command:
      - "--config.file=/etc/prometheus/prometheus.yml"
    extra_hosts:
      - "host.docker.internal:host-gateway"
    networks:
      - monitoring

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    restart: always
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana-storage:/var/lib/grafana
    networks:
      - monitoring

  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    restart: always
    ports:
      - "9100:9100"
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - "--path.procfs=/host/proc"
      - "--path.sysfs=/host/sys"
      - "--path.rootfs=/rootfs"
      - "--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)"
    networks:
      - monitoring

networks:
  monitoring:

volumes:
  grafana-storage:
```

### Key decisions explained

| Setting | Why |
|--------|-----|
| `extra_hosts: host.docker.internal:host-gateway` | On Linux, `host.docker.internal` does not resolve automatically like on Mac/Windows. This line adds it manually so Prometheus can reach the Docker daemon metrics on port `9323`. |
| `/proc`, `/sys`, `/` mounted as `:ro` | Node Exporter needs to read kernel data from the host. The `:ro` flag makes the mounts read-only so the container can never write to the host filesystem. |
| `collector.filesystem.mount-points-exclude` | Filters out virtual kernel filesystems so disk usage metrics only reflect real drives. |

---

## Step 3 — Enable Docker daemon metrics

The `docker` job in Prometheus scrapes Docker engine metrics from port `9323`. This endpoint is disabled by default on Linux and must be enabled manually.

On your VPS, edit `/etc/docker/daemon.json`:

```bash
nano /etc/docker/daemon.json
```

Add the following content:

```json
{
  "metrics-addr": "0.0.0.0:9323",
  "experimental": true
}
```

Then restart the Docker daemon:

```bash
systemctl restart docker
```

---

## Step 4 — Deploy to the VPS

Transfer the `monitoring/` folder to your VPS using the Termius SFTP panel (drag and drop the folder to `/root/monitoring`), then start the stack:

```bash
cd /root/monitoring
docker compose up -d
```

---

## Step 5 — Verify everything is running

**Check that all containers are up:**

```bash
docker compose ps
```

**Check which ports are listening:**

```bash
ss -tlnp
```

You should see ports `9090`, `9100`, `3000`, and `9323` all listed.

**Check Prometheus targets:**

Open `http://<your-vps-ip>:9090/targets` in your browser. All three jobs — `prometheus`, `docker`, and `node-exporter` — should show status **UP**.

**Check Node Exporter directly:**

```bash
curl http://localhost:9100/metrics
```

You should see a large list of metrics prefixed with `node_`.

---

## Step 6 — Connect Grafana to Prometheus

1. Open Grafana at `http://<your-vps-ip>:3000` and log in with `admin / admin`.
2. Go to **Connections → Data sources → Add data source**.
3. Select **Prometheus**.
4. Set the URL to `http://prometheus:9090`.
5. Click **Save & test** — it should confirm the connection is working.

From here you can import a pre-built Node Exporter dashboard from [grafana.com/dashboards](https://grafana.com/grafana/dashboards/) (dashboard ID `1860` is a popular choice) or build your own.

---

## Troubleshooting

**`host.docker.internal` does not resolve**

This happens on Linux when `extra_hosts` is missing from the Prometheus service. Add it to `docker-compose.yml` as shown in Step 2 and run `docker compose up -d` again.

**Port `9323` connection refused**

The Docker daemon metrics endpoint is not enabled. Follow Step 3 to edit `daemon.json` and restart Docker.

**A target shows DOWN in Prometheus**

Run `docker compose ps` to check all containers are running, then verify the service name in `prometheus.yml` matches exactly the service name in `docker-compose.yml`.