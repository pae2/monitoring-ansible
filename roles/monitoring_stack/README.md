# monitoring_stack

Deploy node-exporter and cAdvisor in Docker containers with iptables IP whitelisting.

## Requirements

- Debian 11/12 or Ubuntu 20.04/22.04 target hosts.
- Collection: `community.docker`.
- Docker SDK for Python (`docker`) is **not** required — the role uses `docker_compose_v2`, which calls the `docker compose` plugin installed on the target.

Install collections from the repo root:

```bash
ansible-galaxy collection install -r requirements.yml
```

## Project Structure

```
mon-role/
├── inventory/
│   └── hosts.ini
├── roles/
│   └── monitoring_stack/
│       ├── defaults/main.yml
│       ├── handlers/main.yml
│       ├── meta/main.yml
│       ├── tasks/
│       │   ├── main.yml
│       │   ├── install_docker.yml
│       │   ├── deploy.yml
│       │   └── firewall.yml
│       ├── templates/compose.yml.j2
│       └── README.md
└── site.yml
```

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `node_exporter_port` | `9100` | Node exporter listen port |
| `node_exporter_version` | `v1.8.2` | Image tag |
| `cadvisor_port` | `8080` | cAdvisor listen port |
| `cadvisor_version` | `v0.49.1` | Image tag |
| `monitoring_network` | `monitoring` | Docker network name |
| `monitoring_compose_dir` | `/opt/monitoring` | Where compose.yml is rendered |
| `node_exporter_allowed_ips` | `["127.0.0.1/32"]` | CIDRs allowed to reach node-exporter |
| `cadvisor_allowed_ips` | `["127.0.0.1/32"]` | CIDRs allowed to reach cAdvisor |
| `firewall_enabled` | `true` | If `false`, the role skips all iptables changes |

## Tags

- `docker` / `install` — Docker engine setup only.
- `deploy` — compose render + container refresh.
- `firewall` — iptables rules only.

## Usage

```bash
ansible-playbook -i inventory/hosts.ini site.yml

# Only refresh containers
ansible-playbook -i inventory/hosts.ini site.yml --tags deploy

# Override allowed IPs from CLI
ansible-playbook -i inventory/hosts.ini site.yml \
  -e '{"node_exporter_allowed_ips":["10.0.0.0/8"],"cadvisor_allowed_ips":["10.0.0.0/8"]}'

# Dry run
ansible-playbook -i inventory/hosts.ini site.yml --check
```

## Notes on the firewall

- `node-exporter` runs with `network_mode: host`, so its port is filtered in the `INPUT` chain. The role inserts an explicit loopback-accept rule so local scrapes keep working regardless of `node_exporter_allowed_ips`.
- `cadvisor` publishes its port through Docker's bridge, so filtering happens in the `DOCKER-USER` chain.
- Rules are persisted via `iptables-persistent` (`netfilter-persistent save`), so they survive reboot.

## License

MIT
