# Ethereum stack (validator + monitoring)

Casual homelab project to spin up an Ethereum validator (Holesky by default) plus monitoring/alerts. It’s small and DIY-friendly, but if your infra is solid you could harden it and run it for business too.

## What’s inside
- `ansible-eth-validator/` – node roles/playbooks (base, execution, consensus, validator, optional monitoring).
- `ansible-eth-monitoring/` – monitoring add-ons (Prometheus, Alertmanager, Telegram, dashboards, version check).
- `inventory/hosts.yml` – shared inventory.
- `group_vars/all.yml` – shared vars (clients, paths, metrics, alerts, tokens).
- `playbooks/site.yml` – runs validator then monitoring in one go.
- `ansible.cfg` – points to the shared inventory and both roles dirs.
- `ansible-eth-*/inventory` and `ansible-eth-*/group_vars` are symlinks to the shared files so you can still run from subfolders without drift.

## Quick start (Ansible)
1) Edit `inventory/hosts.yml` with your host/IP, SSH user, and sudo settings.  
2) Edit `group_vars/all.yml` to fit your network, client choices, paths, JWT location, metrics binds, and alert/Telegram settings.  
3) Run the whole stack:
```bash
ansible-playbook playbooks/site.yml
```

Want only one piece?
- Validator only (includes base/execution/consensus/validator; monitoring if `monitoring_enabled: true`):
```bash
ansible-playbook ansible-eth-validator/playbooks/site.yml
```
- Monitoring add-ons only (Prometheus/Alertmanager/Telegram/dashboards) assuming metrics are exposed:
```bash
ansible-playbook ansible-eth-monitoring/playbooks/site.yml
```
<img width="1687" height="695" alt="image" src="https://github.com/user-attachments/assets/adb49d2e-d98b-49f6-b913-b112a1c5b8b7" />
<img width="1692" height="754" alt="image" src="https://github.com/user-attachments/assets/fa486461-6d07-44bf-822d-bc6ad075b9c6" />


### VM / host requirements (homelab baseline)
- Linux x86_64 with systemd, SSH, and sudo.
- CPU: 4 vCPU (8+ if you plan to scale or run mainnet/business use).
- RAM: 16 GB minimum; 32 GB recommended for comfort.
- Disk: SSD/NVMe, at least 1 TB for execution + consensus (more for mainnet).
- Network: stable uplink, open ports for P2P (default 30303 exec, 9000/9001 consensus), and whatever binds you expose for metrics/UI (Prometheus/Grafana).

### Parameters to tweak
- Network: `eth_network` (holesky → sepolia/mainnet), `consensus_p2p_port`, `nethermind_p2p_port`.
-/ Paths: `ethereum_base_dir`, `prometheus_config_path`, `prometheus_rules_dir`.
+ Paths: `ethereum_base_dir`, `prometheus_config_path`, `prometheus_rules_dir`.
- Access: `monitoring_prometheus_listen_address`, `monitoring_grafana_domain`, `monitoring_grafana_root_url`, any firewall rules.
- Clients: `execution_client`, `consensus_client`, `validator_client`, versions/checksums if you pin them.
- Secrets: replace `monitoring_grafana_admin_password`, Telegram tokens, JWT file contents—use Vault/extra-vars for production.

### Upgrades
Use `ansible-eth-monitoring/playbooks/upgrade.yml` to bump Nethermind/Lighthouse binaries. Set `nethermind_download_url` and `lighthouse_download_url`, or let it pull “latest” from GitHub.

### What this stack actually spins up
- Runs execution + consensus + validator services (Holesky by default).
- Automates version checks so you know when new client builds drop.
- Pushes alerts to Telegram (wire up your own bot token/chat id in `group_vars/all.yml`).
- Stands up a light Grafana + Prometheus setup for basic monitoring/alerts.

## Notes
- Secrets in `group_vars/all.yml` are placeholders—swap them before you ever run this outside the lab.
- Root-level `group_vars`/`inventory` keeps both projects in sync; avoid duplicating vars elsewhere.
- If you expose services externally, lock down binds, TLS, and firewall rules first.
