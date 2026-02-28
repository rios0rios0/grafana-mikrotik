# Contributing

Contributions are welcome. By participating, you agree to maintain a respectful and constructive environment.

For coding standards, testing patterns, architecture guidelines, commit conventions, and all
development practices, refer to the **[Development Guide](https://github.com/rios0rios0/guide/wiki)**.

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/) 20.10+
- [Docker Compose](https://docs.docker.com/compose/install/) v2+
- [Git](https://git-scm.com/downloads) 2.30+
- A MikroTik RouterOS v7 device (or emulator) with SNMP enabled for end-to-end testing

## Development Workflow

1. Fork and clone the repository
2. Create a branch: `git checkout -b feat/my-change`
3. Configure the target MikroTik IP address in `prometheus/prometheus.yml`:
   ```bash
   # Edit the target IP (default: 192.168.88.1) in the Mikrotik job section
   vi prometheus/prometheus.yml
   ```
4. Set your current user for the Prometheus container volume permissions:
   ```bash
   sed -i "s/^CURRENT_USER=.*/CURRENT_USER=$(id -u):$(id -g)/" .env
   ```
5. Start the full monitoring stack:
   ```bash
   docker compose up -d
   ```
6. Verify the services are running:
   ```bash
   # Grafana: http://localhost:3000 (admin/mikrotik)
   # Prometheus: http://localhost:9090/targets
   # SNMP Exporter: http://localhost:9116
   docker compose ps
   ```
7. Stop the services when done:
   ```bash
   docker compose down
   ```
8. Commit following the [commit conventions](https://github.com/rios0rios0/guide/wiki/Life-Cycle/Git-Flow)
9. Open a pull request against `main`
