# Ethereum stack validator and monitoring

I made this as a small homelab project to run an Ethereum validator with monitoring and alerts. It is simple and DIY friendly, but if your infra is solid you can harden it for more serious use. I write it in a human way because I use it by myself and share with friends, so it feels more personal.

## What is inside
1. `ansible-eth-validator/` for node roles and playbooks like base, execution, consensus, validator and optional monitoring.
2. `ansible-eth-monitoring/` for monitoring add ons like Prometheus, Alertmanager, Telegram, dashboards and version checks.
3. `inventory/hosts.yml` as the shared inventory.
4. `group_vars/all.yml` as the shared vars for clients, paths, metrics, alerts and tokens.
5. `playbooks/site.yml` to run validator and monitoring in one go.
6. `ansible.cfg` that points to the shared inventory and both roles dirs.
7. `ansible-eth-*/inventory` and `ansible-eth-*/group_vars` are symlinks to the shared files so you can still run from subfolders without drift.

## Quick start with Ansible
1. Edit `inventory/hosts.yml` with your host or IP, SSH user, and sudo settings.
2. Edit `group_vars/all.yml` to fit your network, client choices, paths, JWT location, metrics binds, and alert or Telegram settings.
3. Run the whole stack:

```bash
ansible-playbook playbooks/site.yml
```

If you want only one part, here is how I do it.

Validator only, includes base, execution, consensus, validator and monitoring if `monitoring_enabled: true`.

```bash
ansible-playbook ansible-eth-validator/playbooks/site.yml
```

Monitoring add ons only, Prometheus, Alertmanager, Telegram and dashboards, assuming metrics are exposed.

```bash
ansible-playbook ansible-eth-monitoring/playbooks/site.yml
```

<img width="1687" height="695" alt="image" src="https://github.com/user-attachments/assets/adb49d2e-d98b-49f6-b913-b112a1c5b8b7" />
<img width="1692" height="754" alt="image" src="https://github.com/user-attachments/assets/fa486461-6d07-44bf-822d-bc6ad075b9c6" />

## VM or host requirements
I use this as a homelab baseline and it work for me.

1. Linux x86_64 with systemd, SSH, and sudo.
2. CPU with 4 vCPU, and 8 or more if you plan to scale or run mainnet or business use.
3. RAM 16 GB minimum, 32 GB recommended for comfort.
4. Disk SSD or NVMe, at least 1 TB for execution and consensus, more for mainnet.
5. Network stable uplink, open ports for P2P by default 30303 exec, 9000 or 9001 consensus, and whatever binds you expose for metrics or UI with Prometheus and Grafana.

## Parameters to tweak
1. Network `eth_network` for holesky, sepolia or mainnet, also `consensus_p2p_port` and `nethermind_p2p_port`.
2. Paths `ethereum_base_dir`, `prometheus_config_path`, `prometheus_rules_dir`.
3. Access `monitoring_prometheus_listen_address`, `monitoring_grafana_domain`, `monitoring_grafana_root_url`, and any firewall rules.
4. Clients `execution_client`, `consensus_client`, `validator_client`, and versions or checksums if you pin them.
5. Secrets replace `monitoring_grafana_admin_password`, Telegram tokens, JWT file contents, use Vault or extra vars for production.

## Upgrades
Use `ansible-eth-monitoring/playbooks/upgrade.yml` to bump Nethermind and Lighthouse binaries. Set `nethermind_download_url` and `lighthouse_download_url`, or let it pull latest from GitHub.

## What this stack spins up
1. Runs execution, consensus and validator services, Holesky by default.
2. Automates version checks so you know when new client builds drop.
3. Pushes alerts to Telegram, wire your own bot token and chat id in `group_vars/all.yml`.
4. Stands up a light Grafana and Prometheus setup for basic monitoring and alerts.

## Notes
1. Secrets in `group_vars/all.yml` are placeholders, swap them before you run this outside the lab.
2. Root level `group_vars` and `inventory` keeps both projects in sync, avoid duplicating vars elsewhere.
3. If you expose services externally, lock down binds, TLS, and firewall rules first. I do this step carefully because is easy to forget.
