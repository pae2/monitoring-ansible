---
# monitoring_stack

Deploy node-exporter and cAdvisor in Docker containers with iptables IP whitelisting.

## Requirements

- Docker installed on target hosts
- Python `docker` SDK on control node

## Project Structure

```
mon-role/
├── inventory/
│   └── hosts.ini          # inventory файл
├── roles/
│   └── monitoring_stack/
│       ├── defaults/
│       │   └── main.yml
│       ├── handlers/
│       │   └── main.yml
│       ├── meta/
│       │   └── main.yml
│       ├── tasks/
│       │   └── main.yml
│       ├── templates/
│       │   └── docker-compose.yml.j2
│       └── README.md
└── site.yml               # плейбук
```

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `node_exporter_port` | `9100` | Node exporter listen port |
| `cadvisor_port` | `8080` | cAdvisor listen port |
| `node_exporter_allowed_ips` | `["127.0.0.1/32"]` | IPs allowed to reach node-exporter |
| `cadvisor_allowed_ips` | `["127.0.0.1/32"]` | IPs allowed to reach cAdvisor |
| `firewall_backend` | `iptables` | Firewall backend |

## Usage

```bash
# Проверка inventory
ansible -i inventory/hosts.ini all --list-hosts

# Запуск плейбука
ansible-playbook -i inventory/hosts.ini site.yml

# С конкретными переменными
ansible-playbook -i inventory/hosts.ini site.yml \
  -e node_exporter_allowed_ips='["10.0.0.0/8"]' \
  -e cadvisor_allowed_ips='["10.0.0.0/8"]'

# Только check mode
ansible-playbook -i inventory/hosts.ini site.yml --check
```

## License

MIT
