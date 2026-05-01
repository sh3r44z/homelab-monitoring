#Homelab Monitoring Stack

A production-inspired observability stack running on a self-hosted Ubuntu server. Built with Docker Compose — fully reproducible, zero vendor lock-in.

![Stack](https://img.shields.io/badge/Prometheus-E6522C?style=flat&logo=prometheus&logoColor=white)
![Grafana](https://img.shields.io/badge/Grafana-F46800?style=flat&logo=grafana&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat&logo=docker&logoColor=white)
![Uptime Kuma](https://img.shields.io/badge/Uptime_Kuma-5CDD8B?style=flat&logoColor=white)

---

## Stack

| Service | Purpose | Port |
|---|---|---|
| [Prometheus](https://prometheus.io/) | Metrics collection & storage | `9090` |
| [Grafana](https://grafana.com/) | Dashboards & visualisation | `3030` |
| [Node Exporter](https://github.com/prometheus/node_exporter) | Host system metrics (CPU, RAM, disk) | `9100` |
| [cAdvisor](https://github.com/google/cadvisor) | Container resource metrics | `8080` |
| [Alertmanager](https://prometheus.io/docs/alerting/latest/alertmanager/) | Alert routing & deduplication | `9093` |
| [Uptime Kuma](https://github.com/louislam/uptime-kuma) | Service uptime monitoring | `3001` |

---

## Architecture

```
                    ┌─────────────────────────────────────────┐
                    │           Ubuntu Home Server            │
                    │                                         │
  ┌──────────────┐  │  ┌─────────────┐   ┌────────────────┐  │
  │ Your Browser │──┼──│   Grafana   │──▶│   Prometheus   │  │
  └──────────────┘  │  │  :3030      │   │   :9090        │  │
                    │  └─────────────┘   └───────┬────────┘  │
                    │                            │ scrapes    │
                    │  ┌─────────────┐   ┌───────▼────────┐  │
                    │  │Uptime Kuma  │   │ Node Exporter  │  │
                    │  │  :3001      │   │   :9100        │  │
                    │  └─────────────┘   └────────────────┘  │
                    │                                         │
                    │  ┌─────────────┐   ┌────────────────┐  │
                    │  │Alertmanager │◀──│   cAdvisor     │  │
                    │  │  :9093      │   │   :8080        │  │
                    │  └──────┬──────┘   └────────────────┘  │
                    │         │                               │
                    └─────────┼───────────────────────────────┘
                              │ webhooks
                              ▼
                    ┌─────────────────┐
                    │   n8n / Slack   │
                    └─────────────────┘
```

---

## Quick Start

**Prerequisites:** Docker, Docker Compose v2

```bash
# 1. Clone the repo
git clone https://github.com/sh3r44z/homelab-monitoring.git
cd homelab-monitoring

# 2. Configure environment
cp .env.example .env
nano .env  # set GRAFANA_PASSWORD and ALERT_WEBHOOK_URL

# 3. Start the stack
make up

# 4. Verify all services are running
make ps
```

**Access:**
- Grafana → http://localhost:3030 (default: `admin` / your password)
- Prometheus → http://localhost:9090
- Uptime Kuma → http://localhost:3001

---

## Alert Rules

Configured in `prometheus/alerts.yml`:

| Alert | Condition | Severity |
|---|---|---|
| `HighCPUUsage` | CPU > 80% for 5 min | warning |
| `HighMemoryUsage` | RAM > 85% for 5 min | warning |
| `DiskSpaceLow` | Disk < 15% for 10 min | critical |
| `ServiceDown` | Any scrape target unreachable for 1 min | critical |

Alerts route via Alertmanager to a webhook (configured for n8n for custom notification logic).

---

## Grafana Dashboards

Dashboards are provisioned automatically from `grafana/dashboards/`. Recommended community dashboards to import:

| Dashboard | ID | Use |
|---|---|---|
| Node Exporter Full | `1860` | Host metrics |
| Docker & system monitoring | `893` | Container metrics |
| Alertmanager | `9578` | Alert overview |

To import: Grafana → Dashboards → Import → enter ID.

---

## Useful Commands

```bash
make up                  # Start all services
make down                # Stop all services
make logs                # Tail all logs
make ps                  # Service status
make pull                # Pull latest images
make reload-prometheus   # Hot-reload Prometheus config
```

---

## Project Structure

```
.
├── docker-compose.yml
├── Makefile
├── .env.example
├── prometheus/
│   ├── prometheus.yml       # Scrape config & alerting rules pointer
│   └── alerts.yml           # Alert rule definitions
├── alertmanager/
│   └── alertmanager.yml     # Routing & receiver config
├── grafana/
│   ├── dashboards/          # Exported dashboard JSON files
│   └── provisioning/
│       ├── datasources/     # Auto-provision Prometheus datasource
│       └── dashboards/      # Auto-provision dashboard loader
└── docs/
    └── architecture.svg     # Architecture diagram
```

---

## Key Design Decisions

- **No auth proxy** — this stack is LAN-only; Authentik SSO is handled at the network layer separately.
- **Remote state not needed** — single-host setup; volume mounts handle persistence.
- **Alertmanager → n8n** — alerts hit an n8n webhook for flexible routing (Slack, email, custom logic) without hardcoding credentials in Alertmanager config.
- **30-day retention** — configured in Prometheus for a balance of history vs. disk usage on a home server.

---

## What I Learned

- Designing a scrape topology and understanding Prometheus' pull model
- Writing PromQL expressions for real alert conditions
- Grafana provisioning via code (no manual dashboard setup)
- Alert deduplication and inhibition rules in Alertmanager
- Integrating alerting with n8n for webhook-based notification workflows

---

## Related Projects

- [homelab-docker-stack](https://github.com/sh3r44z/homelab-docker-stack) — the broader home server setup this monitoring stack observes
- [learn-app](https://github.com/sh3r44z/learn-app) — personal study tool (also monitored by this stack)

---

*Part of my homelab & DevOps portfolio. Server: Ubuntu 22.04 · Docker 24 · 192.168.0.10*
