# debian-server-role

Ansible role for provisioning a Debian server with security hardening and essential services.

## What it does

| Component | Description |
|-----------|-------------|
| System upgrade | Full APT upgrade + installs `rsync` and `sudo` |
| User | Creates a non-root user with sudo and Docker access, with a password and/or SSH keys |
| sshd | Optional hardening: key-only login for that user, no root, no passwords |
| Firewall | iptables default-deny policy, allowing only loopback, established connections, and SSH |
| fail2ban | SSH brute-force protection via systemd journal |
| Postfix | SMTP relay (e.g. Gmail) over TLS, with SASL authentication or authorized by IP; listens on loopback only |
| Docker | Installs Docker CE and the compose plugin from Docker's repository; optional `daemon.json` |
| Unattended upgrades | Automatic Debian security updates with email notifications; configurable origins and automatic reboot |

## Requirements

- Ansible >= 2.9
- Python 3 on the control node
- SSH access as `root` to the target host, or as a sudo user when re-running after `sshd_hardening`
- A Debian-based target system (tested on Debian 11/12)

Install required collections:

```bash
ansible-galaxy collection install -r requirements.yml
```

## Usage

```bash
ansible-playbook debian-server.yml
```

The playbook will prompt for the target host IP and all sensitive configuration values at runtime.

The role can also be used from another playbook with the variables below set in
the inventory or `vars_files` (e.g. a key-only server behind an IP-authorized
relay: `user_ssh_authorized_keys`, `sshd_hardening: true`, `postfix_relay_sasl: false`).

## Variables

All variables have defaults in `roles/debian-server-role/defaults/main.yml`. Values provided via `vars_prompt` at runtime override these defaults.

| Variable | Default | Description |
|----------|---------|-------------|
| `user_name` | `ds` | Username for the non-root user |
| `user_password` | *(prompted)* | User login password; empty creates the account without one |
| `user_ssh_authorized_keys` | `[]` | Public keys allowed to log in as the user; when set, they replace any other |
| `sshd_hardening` | `false` | Key-only login, only `user_name`, no root, no passwords. Requires `user_ssh_authorized_keys` |
| `docker_daemon_options` | `{}` | Written to `/etc/docker/daemon.json` when not empty |
| `fail2ban_ssh_bantime` | `3600` | SSH ban duration in seconds |
| `fail2ban_ssh_findtime` | `600` | Window for counting failed attempts, in seconds |
| `fail2ban_ssh_max_retries` | `3` | Failed attempts before ban |
| `postfix_fqdn` | `localhost` | Fully qualified domain name for Postfix |
| `postfix_inet_interfaces` | `loopback-only` | Interfaces Postfix listens on |
| `postfix_relay_host` | `smtp.gmail.com` | SMTP relay host |
| `postfix_relay_port` | `587` | SMTP relay port |
| `postfix_relay_mail` | `user@example.com` | Sender address used in emails |
| `postfix_relay_sasl` | `true` | Authenticate to the relay; `false` for a relay that authorizes by IP |
| `postfix_relay_username` | `user@example.com` | SMTP authentication username |
| `postfix_relay_password` | *(prompted)* | SMTP authentication password |
| `unattended_upgrades_email` | `user@example.com` | Email for upgrade notifications |
| `unattended_upgrades_origins` | Debian security | `Origins-Pattern` entries, e.g. add `origin=Docker,suite=${distro_codename}` |
| `unattended_upgrades_automatic_reboot` | `false` | Reboot when an update requires it |
| `unattended_upgrades_automatic_reboot_time` | `04:30` | When that reboot happens |

## Security notes

- SASL credentials are written to `/etc/postfix/sasl/` with mode `0600` and never logged; with `postfix_relay_sasl: false` nothing is stored.
- Passwordless sudo is granted to the `%sudo` group.
- iptables rules are saved immediately after application via `netfilter-persistent`.
- The `user_password` and SASL password are protected with `no_log: true` throughout.
- `sshd_hardening` is applied last in the SSH session that runs the role; the next run must connect as `user_name` with one of the authorized keys.

## License

MIT
