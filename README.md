<h1 align="center">Grafana Mikrotik</h1>
<p align="center">
    <a href="https://github.com/rios0rios0/grafana-mikrotik/releases/latest">
        <img src="https://img.shields.io/github/release/rios0rios0/grafana-mikrotik.svg?style=for-the-badge&logo=github" alt="Latest Release"/></a>
    <a href="https://github.com/rios0rios0/grafana-mikrotik/blob/main/LICENSE">
        <img src="https://img.shields.io/github/license/rios0rios0/grafana-mikrotik.svg?style=for-the-badge&logo=github" alt="License"/></a>
</p>

![mikrotikOS](https://img.shields.io/badge/Mikrotik_ROS-v7.3.1-blue)
![Grafana](https://img.shields.io/badge/Grafana-v9.0.0-orange?logo=grafana)
![Prometheus](https://img.shields.io/badge/Prometheus-v2.36.0-red?logo=prometheus)
![snmp_exporter](https://img.shields.io/badge/snmp__exporter-v0.20.0-red?logo=prometheus)

A Docker Compose stack for monitoring MikroTik RouterOS v7 devices using Prometheus SNMP exporter and Grafana dashboards.

## Deploy with Docker Compose

### Deploy with Bash Script

```console
curl -fsSL https://raw.githubusercontent.com/IgorKha/Grafana-Mikrotik/master/run.sh | bash -s -- --config
```

```console
  You can also pass some arguments to script to set some these options:

    --config: change the user and password to grafana and specify the mikrotik IP address

    --stop: stop docker containers

    --help
```

For example:

```console
    bash run.sh --config
```

[![asciicast](https://asciinema.org/a/nOhuc7LvI6bRWbg7dcvqFQ4Kc.png)](https://asciinema.org/a/nOhuc7LvI6bRWbg7dcvqFQ4Kc)

### Deploy with Docker Compose Manually

1. Change targets IP (192.168.88.1) in the file `prometheus/prometheus.yml`

2. Run:

```console
docker-compose up -d
```

3. Open [localhost:3000](http://localhost:3000)

- Grafana login: `admin`
- Password: `mikrotik`

If you want to change the credentials, edit the `.env` file.

## Manual Deploy

1. Add into `prometheus.yml`:

```yml
  - job_name: Mikrotik
    static_configs:
      - targets:
        - 192.168.88.1  # SNMP device IP.
    metrics_path: /snmp
    params:
      module: [mikrotik]
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: localhost:9116  # The SNMP exporter's real hostname:port.
```

2. Configure Prometheus and run `/snmp/snmp_exporter`

3. Add dashboard: <https://grafana.com/grafana/dashboards/14420>

## Docker snmp_exporter

[![Docker Pulls](https://img.shields.io/docker/pulls/mashinkopochinko/snmp_exporter_mikrotik?logo=docker)](https://hub.docker.com/repository/docker/mashinkopochinko/snmp_exporter_mikrotik)

> amd64-linux container

```console
sudo docker run -d -p 9116:9116 mashinkopochinko/snmp_exporter_mikrotik:latest
```

![img1](/readme/screen.png)

## Contributing

Contributions are welcome. See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## License

This project is licensed under the terms specified in the [LICENSE](LICENSE) file.
