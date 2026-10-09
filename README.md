# debian-server-role

Ansible role for provisioning a Debian server with security hardening and essential services.

## What it does

| Component | Description |
|-----------|-------------|
| System upgrade | Full APT upgrade + installs `rsync` and `sudo` |
| User | Creates a non-root user with sudo and Docker access |
| Firewall | iptables default-deny policy, allowing only loopback, established connections, and SSH |
| fail2ban | SSH brute-force protection via systemd journal |
| Postfix | SMTP relay (e.g. Gmail) with SASL/TLS authentication |
| Docker | Installs `docker.io` and `docker-compose` |
| Unattended upgrades | Automatic Debian security updates with email notifications |

## Requirements

- Ansible >= 2.9
- Python 3 on the control node
- SSH access as `root` to the target host
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

## Variables

All variables have defaults in `roles/debian-server-role/defaults/main.yml`. Values provided via `vars_prompt` at runtime override these defaults.

| Variable | Default | Description |
|----------|---------|-------------|
| `user_name` | `ds` | Username for the non-root user |
| `user_password` | *(prompted)* | User login password |
| `fail2ban_ssh_bantime` | `3600` | SSH ban duration in seconds |
| `fail2ban_ssh_max_retries` | `3` | Failed attempts before ban |
| `postfix_fqdn` | `localhost` | Fully qualified domain name for Postfix |
| `postfix_relay_host` | `smtp.gmail.com` | SMTP relay host |
| `postfix_relay_port` | `587` | SMTP relay port |
| `postfix_relay_mail` | `user@example.com` | Sender address used in emails |
| `postfix_relay_username` | `user@example.com` | SMTP authentication username |
| `postfix_relay_password` | *(prompted)* | SMTP authentication password |
| `unattended_upgrades_email` | `user@example.com` | Email for upgrade notifications |

## Security notes

- SASL credentials are written to `/etc/postfix/sasl/` with mode `0600` and never logged.
- Passwordless sudo is granted to the `%sudo` group.
- iptables rules are saved immediately after application via `netfilter-persistent`.
- The `user_password` and SASL password are protected with `no_log: true` throughout.

## License

MIT
