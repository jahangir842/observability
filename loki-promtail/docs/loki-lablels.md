# ✅ What Are Loki Labels?

Labels are **key-value pairs** attached to logs.
They help you **query, group, and filter** logs.

Typical useful labels for microservices:

| Label     | Meaning                              |
| --------- | ------------------------------------ |
| `app`     | Name of the entire application       |
| `service` | Which microservice produced this log |
| `tier`    | Frontend / backend / database tier   |
| `env`     | Environment (dev, staging, prod)     |
| `version` | Optional service version             |
| `team`    | Optional ownership                   |

---

# ✅ Example: Docker Compose with Loki Labels

Below is a clean example showing how to label your three services.

```yaml
version: "3.9"

services:
  frontend:
    image: myapp/frontend:latest
    container_name: frontend
    ports:
      - "3000:3000"
    labels:
      logging: "loki"
      loki.app: "myapp"
      loki.service: "frontend"
      loki.tier: "web"
      loki.env: "dev"

  backend:
    image: myapp/backend:latest
    container_name: backend
    ports:
      - "8080:8080"
    labels:
      logging: "loki"
      loki.app: "myapp"
      loki.service: "backend"
      loki.tier: "api"
      loki.env: "dev"

  database:
    image: postgres:15
    container_name: db
    environment:
      POSTGRES_USER: "admin"
      POSTGRES_PASSWORD: "admin123"
    labels:
      logging: "loki"
      loki.app: "myapp"
      loki.service: "database"
      loki.tier: "db"
      loki.env: "dev"
```

---

# 🔥 How Promtail Reads These Labels

In your Promtail config:

```yaml
scrape_configs:
  - job_name: docker
    docker_sd_configs:
      - host: unix:///var/run/docker.sock
    relabel_configs:
      - source_labels: ['__meta_docker_container_label_loki_service']
        target_label: "service"
      - source_labels: ['__meta_docker_container_label_loki_tier']
        target_label: "tier"
      - source_labels: ['__meta_docker_container_label_loki_app']
        target_label: "app"
      - source_labels: ['__meta_docker_container_label_loki_env']
        target_label: "env"
```

✔️ This automatically attaches your compose labels as Loki log labels.

---

# 🧩 Best Practice Label Strategy

### Use consistent naming:

```
app=myapp
service=frontend | backend | database
tier=web | api | db
env=dev | staging | prod
```

### DO NOT use dynamic labels

(e.g. timestamps, random strings) — they explode your Loki index.

---

# 🟢 Querying in Loki / Grafana

Examples:

### See backend API logs

```
{service="backend"}
```

### See database logs only

```
{tier="db"}
```

### See everything in dev

```
{env="dev"}
```

### See all logs for the entire application

```
{app="myapp"}
```

---

# 🧰 Pro Tip: Minimal Label Set

If you want it super simple:

```yaml
labels:
  loki.service: "backend"
```

Then in Grafana:

```
{service="backend"}
```

---

# 🎯 Final Recommendations

* Use **2–4 stable labels max**
* Keep them **consistent across all services**
* Label for **WHO** produced the log (service name) not WHAT’s in the log
* Store labels in **docker compose**, not in code

---

