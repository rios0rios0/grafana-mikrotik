# Copilot Instructions

## Project Overview

**grafana-mikrotik** is a Docker Compose monitoring stack for MikroTik RouterOS v7 devices.
It collects device metrics via SNMP, scrapes them with the Prometheus SNMP exporter, stores
them in Prometheus, and visualises them in pre-provisioned Grafana dashboards.

## Project Structure

```
.
├── docker-compose.yml           # Defines the three-service monitoring stack
├── run.sh                       # Interactive Bash helper: clone, configure, start/stop the stack
├── prometheus/
│   ├── prometheus.yml           # Scrape config; the MikroTik target IP is set here
│   └── data/                    # Persistent Prometheus TSDB volume (bind-mounted)
├── grafana/
│   └── provisioning/
│       ├── datasources/
│       │   └── datasource.yml   # Auto-provisions the Prometheus data source
│       └── dashboards/
│           ├── dashboard.yml    # Dashboard provider config
│           └── Mikrotik-snmp-prometheus.json  # Pre-built MikroTik dashboard
├── snmp/
│   ├── snmp.yml                 # SNMP exporter module definition for MikroTik
│   └── Dockerfile               # Builds a custom snmp_exporter image (amd64-linux)
├── .env                         # CURRENT_USER (UID:GID for Prometheus volume ownership)
├── .grafana                     # Grafana admin credentials and sign-up flag
├── .prometheus                  # MIKROTIK_IP used by run.sh to patch prometheus.yml
└── .github/
    └── workflows/
        └── action.yml           # CI: lint (Shellcheck) → test (deploy + health checks)
```

## Build & Development Commands

### Prerequisites

- Docker 20.10+
- Docker Compose v2+
- Git 2.30+

### Start the stack

```bash
# Interactive setup (prompts for MikroTik IP and Grafana credentials)
bash run.sh --config

# Or manually
docker compose up -d
```

### Stop the stack

```bash
docker compose down
# or
bash run.sh --stop
```

### Manual configuration

```bash
# 1. Set the MikroTik target IP in Prometheus config
vi prometheus/prometheus.yml   # change the target under the Mikrotik job

# 2. Fix volume ownership for the Prometheus container
sed -i "s/^CURRENT_USER=.*/CURRENT_USER=$(id -u):$(id -g)/" .env

# 3. (Optional) Change Grafana credentials
vi .grafana   # GF_SECURITY_ADMIN_USER and GF_SECURITY_ADMIN_PASSWORD
```

### Verify running services

| Service       | URL                              | Default credentials |
|---------------|----------------------------------|---------------------|
| Grafana       | <http://localhost:3000>          | admin / mikrotik    |
| Prometheus    | <http://localhost:9090/targets>  | —                   |
| SNMP Exporter | <http://localhost:9116>          | —                   |

## Architecture

The stack follows a straightforward metrics-pipeline pattern:

```
MikroTik device (SNMP)
        │  UDP 161
        ▼
  snmp_exporter  :9116   ← scrapes SNMP OIDs defined in snmp/snmp.yml
        │  HTTP /snmp
        ▼
    Prometheus   :9090   ← stores time-series; scrape interval 15 s
        │
        ▼
     Grafana     :3000   ← reads Prometheus via auto-provisioned datasource
```

- **Grafana** (`grafana/grafana:9.0.0`) — dashboards and visualisation.  
  Provisioning is fully declarative: datasources and dashboards are loaded from
  `grafana/provisioning/` at startup; no manual UI configuration is needed.
- **Prometheus** (`prom/prometheus`) — metrics storage and scraping.  
  The container runs as the host user (`CURRENT_USER`) to avoid volume permission issues.
- **SNMP Exporter** (`prom/snmp-exporter`) — translates SNMP OIDs to Prometheus metrics.  
  The MikroTik-specific module is defined in `snmp/snmp.yml`.

## Coding Conventions

- **Bash (`run.sh`)**: follow existing style — `set -e`, named functions, colour variables,
  `ask()` helper for yes/no prompts, `fmt_error()` for error messages.
- **YAML indentation**: 2 spaces throughout (Docker Compose, Prometheus, Grafana provisioning).
- **Environment variables**: sensitive/runtime values live in dot-files (`.grafana`,
  `.prometheus`, `.env`), never hard-coded in `docker-compose.yml`.
- **Container names**: prefixed with `mk_` (e.g., `mk_grafana`, `mk_prometheus`,
  `mk_snmp_exporter`) to avoid collisions with other local containers.
- Commit messages and branching follow the
  [rios0rios0/guide Git-Flow conventions](https://github.com/rios0rios0/guide/wiki/Life-Cycle/Git-Flow).

## Testing

There is no unit-test suite. Validation is integration-based:

1. Deploy the stack with `docker compose up -d`.
2. Wait for services to become healthy, then curl the health endpoints:
   ```bash
   curl -GLsS --retry 5 --retry-delay 2 "http://localhost:3000/api/health"
   curl -GLsS --retry 5 --retry-delay 2 "http://localhost:9090/-/ready"
   curl -ILsS --retry 5 --retry-delay 2 "http://localhost:9116"
   ```
3. For end-to-end coverage, a real (or emulated) MikroTik RouterOS v7 device with SNMP
   enabled is required.

## CI/CD

The GitHub Actions pipeline (`.github/workflows/action.yml`) runs on every push and pull
request with two sequential jobs:

| Job    | What it does |
|--------|--------------|
| `lint` | Runs [Shellcheck](https://github.com/koalaman/shellcheck) on `run.sh` at `warning` and `error` severity. |
| `test` | Deploys the stack via `run.sh`, then health-checks Grafana, Prometheus, and the SNMP exporter. |

There are no local `make` targets. To replicate the CI checks locally:

```bash
# Lint
shellcheck -S warning run.sh

# Integration test
bash run.sh
sleep 5
curl -GLsS "http://localhost:3000/api/health"
curl -GLsS "http://localhost:9090/-/ready"
curl -ILsS "http://localhost:9116"
```
