## Adding MySQL and PostgreSQL to the monitoring stack

This guide extends the base monitoring stack (Prometheus + Grafana + Node Exporter) to include MySQL and PostgreSQL metrics. The approach follows the same exporter pattern — a dedicated exporter per database translates DB internals into Prometheus metrics.

## Architecture overview

```
mysql-container (:3306) -> mysql-exporter (:9104) -> Prometheus (:9090) -> Grafana (:3000)
instance-prostgress (:5432) -> postgres-exporter (:9187) ----------------------->|
```

---

## Prerequisites

- The base monitoring stack is already running (Prometheus, Grafana, Node Exporter).
- MySQL and PostgreSQL are running as Docker containers on the same server.
- Both DB containers must be connected to the same Docker network as the monitoring stack.

---

## Step 1 — Connect DB containers to the monitoring network

Containers started outside of Docker Compose need to be manually added to the monitoring network.

> **Note:** Docker Compose prefixes network names with the project folder name. If your stack lives in `/root/monitoring`, the network will be called `monitoring_monitoring`.

```bash
docker network connect monitoring_monitoring mysql-container
docker network connect monitoring_monitoring instance-prostgress
```

Verify they appear in the network:

```bash
docker network inspect monitoring_monitoring | grep Name
```

---

## Step 2 — Create read-only exporter users

Never use root or your application user for exporters. Create dedicated read-only users on each database.

**MySQL:**

```bash
docker exec mysql-container mysql -u root -pmypassword -e "CREATE USER 'exporter'@'%' IDENTIFIED BY 'exporter123'; GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'exporter'@'%'; FLUSH PRIVILEGES;"
```

**PostgreSQL:**

```bash
docker exec instance-prostgress psql -U postgres -c "CREATE USER exporter WITH PASSWORD 'exporter123'; GRANT pg_monitor TO exporter;"
```

---

## Step 3 — Create the MySQL exporter config file

Version `0.19.0` of `mysqld-exporter` requires credentials via a `.my.cnf` config file instead of environment variables.

```bash
mkdir -p /root/monitoring/mysql-exporter
```

Create `/root/monitoring/mysql-exporter/.my.cnf` with the following content:

```ini
[client]
user=exporter
password=exporter123
host=mysql-container
port=3306
```

---

## Step 4 — Add exporters to docker-compose.yml

Add both exporter services to your existing `docker-compose.yml`:

```yaml
  mysql-exporter:
    image: prom/mysqld-exporter:latest
    container_name: mysql-exporter
    restart: always
    ports:
      - "9104:9104"
    volumes:
      - ./mysql-exporter/.my.cnf:/cfg/.my.cnf:ro
    command:
      - "--config.my-cnf=/cfg/.my.cnf"
      - "--collect.info_schema.innodb_metrics"
    networks:
      - monitoring

  postgres-exporter:
    image: prometheuscommunity/postgres-exporter:latest
    container_name: postgres-exporter
    restart: always
    ports:
      - "9187:9187"
    environment:
      - DATA_SOURCE_NAME=postgresql://exporter:exporter123@instance-prostgress:5432/postgres?sslmode=disable
    networks:
      - monitoring
```

### Key decisions explained

| Setting | Why |
|---------|-----|
| `.my.cnf` mounted as `:ro` | Version 0.19.0+ of mysqld-exporter dropped `DATA_SOURCE_NAME` support and requires a config file. The `:ro` flag prevents the container from modifying it. |
| `--collect.info_schema.innodb_metrics` | Enables InnoDB metrics like buffer pool size, disabled by default but required by most Grafana dashboards. |
| `sslmode=disable` | Disables SSL for internal Docker network connections where encryption is not needed. |

---

## Step 5 — Add scrape jobs to prometheus.yml

```yaml
  - job_name: 'mysql'
    static_configs:
      - targets: ['mysql-exporter:9104']

  - job_name: 'postgres'
    static_configs:
      - targets: ['postgres-exporter:9187']
```

---

## Step 6 — Deploy and verify

Bring up the new services and reload Prometheus:

```bash
cd /root/monitoring
docker compose up -d
docker compose restart prometheus
```

Check all exporters are running:

```bash
docker ps | grep exporter
```

Verify metrics are flowing in Prometheus by searching for `mysql_up` (should return `1`) and `pg_up` (should return `1`).

---

## Step 7 — Import dashboards in Grafana

Go to **Dashboards → Import** and use these community dashboard IDs:

| Database | Dashboard ID |
|----------|-------------|
| MySQL | `7362` |
| PostgreSQL | `9628` |

Select your Prometheus data source when prompted.

---

## Troubleshooting

**`Access denied for user 'exporter'`**

The exporter user password doesn't match what's in `.my.cnf`. Drop and recreate the user:

```bash
docker exec mysql-container mysql -u root -pmypassword -e "DROP USER 'exporter'@'%'; CREATE USER 'exporter'@'%' IDENTIFIED BY 'exporter123'; GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'exporter'@'%'; FLUSH PRIVILEGES;"
```

**`no configuration found` in mysql-exporter logs**

The `.my.cnf` file is not being picked up. Make sure the `--config.my-cnf` flag in the `command` block points to the correct mounted path:

```yaml
command:
  - "--config.my-cnf=/cfg/.my.cnf"
```

**Panel shows "No data" in Grafana**

Some panels join MySQL and Node Exporter metrics using the `instance` label. If the instances don't match, the query returns nothing. Edit the panel and replace `on (instance)` with `on ()` to remove the label matching requirement:

```promql
(mysql_global_variables_innodb_buffer_pool_size{instance="$host"} * 100) / on () node_memory_MemTotal_bytes
```

**Target shows DOWN in Prometheus**

Check that the DB container is on the monitoring network:

```bash
docker network inspect monitoring_monitoring | grep Name
```

If the container is missing, connect it manually:

```bash
docker network connect monitoring_monitoring mysql-container
```