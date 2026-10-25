# Trading Stats Dashboard

A self-hosted trading performance dashboard for a fresh Ubuntu VPS. It receives heartbeat, signal, trade and position data, then displays live statistics, archived reports and a responsive profit and loss calendar.

## What it includes

- Live account balance and account status
- Data source and EA heartbeat status
- Open-trade monitoring
- Six-card dashboard layout
- Current week and month statistics
- Full weekly report
- Archived weekly and monthly reports
- Responsive profit and loss calendar
- Signal, position and completed-trade API endpoints
- SQLite data storage
- Nginx reverse proxy
- systemd service with automatic restart
- Local reset, new-week and new-month helpers
- Automatic backup and restoration of an existing statistics database during reinstall

## Requirements

- Fresh or compatible Ubuntu VPS
- Root or sudo access
- Public IPv4 address
- TCP ports `22` and `80` available
- At least eight characters for the dashboard secret

## Fresh Ubuntu installation

Run:

```bash
{ command -v curl >/dev/null 2>&1 || { sudo apt-get update -y && sudo apt-get install -y curl; }; } && curl -fsSL https://raw.githubusercontent.com/waqaarhussain/trading-stats-dashboard/main/install.sh -o /tmp/trading-stats-dashboard-install.sh && sudo bash /tmp/trading-stats-dashboard-install.sh
```

The installer asks for the dashboard secret through a hidden terminal prompt. Nothing appears while the secret is being typed. Press Enter when finished.

The secret must contain between 8 and 128 characters and may use letters, numbers, dots, underscores and hyphens.

## What the installer does

1. Installs Python, Nginx, SQLite, curl and UFW.
2. Allows OpenSSH and TCP port `80`.
3. Detects the VPS public IPv4 address.
4. Creates the Python application under `/opt/waqaar-stats-dashboard`.
5. Stores the dashboard secret in a root-readable environment file.
6. Creates or restores the SQLite statistics database.
7. Installs and starts the `waqaar-stats-dashboard` systemd service.
8. Configures Nginx as the public reverse proxy.
9. Installs the local helper commands.
10. Validates the dashboard pages, layout and required database tables.

If an existing database is present, it is backed up before the application directory is rebuilt and restored afterwards.

## Dashboard addresses

The installer prints the exact VPS address when it finishes.

| Address | Purpose |
| --- | --- |
| `http://VPS-IP/` | Main trading dashboard |
| `http://VPS-IP/health` | Health check |
| `http://VPS-IP/thisweek` | Current week's full report |
| `http://VPS-IP/allhistory` | Archived weekly and monthly reports |

It also creates an `sslip.io` hostname from the public IP for Nginx host matching.

## Helper commands

Reset all stored dashboard statistics:

```bash
sudo reset
```

Archive the current week and start a new week:

```bash
sudo new week
```

Archive the current month and start a new month:

```bash
sudo new month
```

The helpers read the dashboard secret from the protected VPS environment file. The secret is not embedded in the public scripts.

## Service management

Check the service:

```bash
sudo systemctl status waqaar-stats-dashboard --no-pager
```

View recent logs:

```bash
sudo journalctl -u waqaar-stats-dashboard -n 100 --no-pager
```

Restart the application:

```bash
sudo systemctl restart waqaar-stats-dashboard
```

Validate Nginx:

```bash
sudo nginx -t
```

## Important locations

| Path | Purpose |
| --- | --- |
| `/opt/waqaar-stats-dashboard` | Application, virtual environment and data |
| `/opt/waqaar-stats-dashboard/.env` | Dashboard configuration and secret |
| `/opt/waqaar-stats-dashboard/data/stats.db` | SQLite statistics database |
| `/etc/systemd/system/waqaar-stats-dashboard.service` | Application service |
| `/etc/nginx/sites-available/waqaar-stats-dashboard` | Nginx site |
| `/usr/local/bin/reset` | Statistics reset helper |
| `/usr/local/bin/new` | Week and month rollover helper |
| `/root/waqaar-stats-db-before-install-*.db` | Pre-install database backups |

## Security

- The public repository contains no dashboard secret.
- The installer collects the secret directly from the VPS terminal.
- The environment file is stored with mode `600`.
- The application listens on `127.0.0.1:8000`; Nginx exposes it on TCP port `80`.
- The current setup uses plain HTTP. Traffic and remote API submissions are not encrypted in transit.
- Use a provider firewall to limit access if the dashboard is not intended to be publicly reachable.
- Back up the SQLite database before deleting or rebuilding the VPS.

## Reinstall behaviour

Running the installer again removes and rebuilds the application directory. It preserves the existing SQLite database by copying it to `/root`, then restores it after installation.

Review the printed backup path and retain an off-VPS copy if the trading history is important.
