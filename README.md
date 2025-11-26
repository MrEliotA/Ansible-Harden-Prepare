# Ansible Harden & Prepare

This project provides an opinionated set of Ansible playbooks and roles to harden fresh servers, install Docker, and bootstrap a self-hosted repository stack (Traefik, Nexus, and supporting services). It is designed for Debian-based hosts and aims to deliver a secure, reproducible baseline with minimal manual effort.

## What you get
- **Server preparation & hardening**: Installs a curated package baseline, applies SSH, firewall, Fail2ban, iptables, sysctl, auditd, journald, DNS, and host file configurations, and downloads Lynis for auditing.
- **Docker installation**: Installs Docker Engine, configures logging and registry mirrors, and installs Docker Compose.
- **Repository stack**: Deploys a repo server with Traefik and Nexus using templated Docker Compose and Traefik configurations.

## Repository layout
- `preparing.yaml` — Playbook to harden hosts and install Docker.
- `repo-server.yaml` — Playbook to deploy the repository stack on prepared hosts.
- `inventory/host.yaml` — Example inventory targeting the `repo-servers` group.
- `inventory/group_vars/all/` — Shared variables for Docker, hardening, proxies, backups, and repo server configuration.
- `roles/preparing_server/` — Hardening tasks, templates (sshd, iptables, limits, sysctl, auditd, DNS, journald, Fail2ban), and Lynis download logic.
- `roles/docker/` — Installs and configures Docker and Docker Compose on Debian-family hosts.
- `roles/repo-server/` — Pulls container images and renders Traefik and Nexus configuration templates.

## Prerequisites
- Ansible control machine with network access to targets.
- Target hosts running a Debian-family distribution with `sudo` access.
- SSH key-based access to the target host (see `inventory/host.yaml`), plus an optional SSH bastion if required.
- Update or encrypt sensitive values with Ansible Vault (for example `inventory/group_vars/all/vault.yaml`).

## Inventory & variables
- **Inventory**: Define hosts under the `repo-servers` group in `inventory/host.yaml`. Example values for host, SSH user, port, and key path are provided.
- **Docker config**: Edit `inventory/group_vars/all/docker.yaml` to set proxy, registry mirror, logging limits, and `no_proxy` values.
- **Hardening config**: Tweak packages, SSH, iptables, Fail2ban, timeout, auditd, DNS, limits, and sysctl settings in `inventory/group_vars/all/preparing.yaml` to match your policies.
- **Repo server config**: Set domain, subdomains, Traefik ACME email, Nexus credentials, and image versions in `inventory/group_vars/all/repo-server.yaml`.
- **Proxy settings**: Provide proxy environment values in `inventory/group_vars/all/proxy.yaml` and reference them where needed.
- **Backups**: Adjust Docker volume and destination paths in `inventory/group_vars/all/backup.yaml` if you enable the commented backup task.
- **Secrets**: Store any passwords or tokens in `inventory/group_vars/all/vault.yaml` and encrypt with `ansible-vault encrypt inventory/group_vars/all/vault.yaml`.

## Running the playbooks
1. Install dependencies on your control machine (e.g., `pip install ansible`).
2. Review and update the inventory and group variables to match your environment.
3. Run the preparation playbook to harden hosts and install Docker:
   ```bash
   ansible-playbook -i inventory/host.yaml preparing.yaml
   ```
4. Deploy the repository stack (Traefik + Nexus) once hosts are prepared:
   ```bash
   ansible-playbook -i inventory/host.yaml repo-server.yaml
   ```

## Useful tags
- `preparing_server`, `install_packages`, `tuning`, `hardening`, `firewall` — hardening scope.
- `docker_setup`, `install_docker`, `config_docker`, `install_compose` — Docker installation and configuration.
- `repo_setup` — repository stack deployment tasks.

## Tips
- Verify that your `ansible_user`, `ansible_port`, and SSH key path in `inventory/host.yaml` are correct before running the playbooks.
- If using HTTP/HTTPS proxies, configure `http_proxy`/`https_proxy` in `inventory/group_vars/all/proxy.yaml` and mirror settings in `inventory/group_vars/all/docker.yaml`.
- Keep your vault file encrypted in version control and decrypt only when editing to avoid leaking secrets.
