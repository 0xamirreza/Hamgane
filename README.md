# HAMGANE

HAMGANE is a single-binary web dashboard for controlled upload traffic in an authorized two-server VPN or lab setup.

> Educational and authorized use only. Run HAMGANE only on servers, IPs, and networks you own or have explicit permission to test.

## Features

- Single binary: dashboard, uploader, receiver, and deploy helper are built into `hamgane`.
- Web dashboard with login, live monitoring, endpoint status, upload progress, and traffic balance.
- Sequential upload logic: sleep window → upload selected size until finished → choose next sleep/size.
- TCP/HTTP receiver deployment over SSH for a foreign/remote server.
- Optional UDP mode for authorized target IP/port tests.
- Schedule support: always run, run once, every day, or selected days.

When `running` is enabled and the schedule allows execution, HAMGANE sends live traffic.

## Scenario

```text
VPN users → Iran/source server (HAMGANE) → Foreign server (receiver) → Internet
                 │
                 └── controlled upload cycles + live TX/RX monitoring
```

- **Source server:** runs `./hamgane` and controls the dashboard.
- **Foreign server:** receives HTTP uploads through the deployed receiver.

## Package Files

The deploy/release bundle should contain only:

```text
hamgane
config.json.example
.env.example
README.md
```

For your own server, create real config files from the examples:

```bash
cp config.json.example config.json
cp .env.example .env
```

## Quick Start

```bash
chmod +x hamgane
cp config.json.example config.json
cp .env.example .env
nano .env
nano config.json
./hamgane validate
./hamgane
```

Open the dashboard:

```text
http://SERVER_IP:9898
```

Login credentials come from `.env`.

## `.env`

`.env` only controls the web dashboard bind address and login. These values are not editable from the dashboard.

```env
HAMGANE_WEB_HOST=127.0.0.1
HAMGANE_WEB_PORT=9898
HAMGANE_WEB_USERNAME=admin
HAMGANE_WEB_PASSWORD=change-this-password
```

Change `HAMGANE_WEB_PASSWORD` before starting. The default password is rejected. Use `127.0.0.1` for local access and `0.0.0.0` only on authorized servers.

## `config.json`

Runtime settings are stored in `config.json` and can be edited from the dashboard.

Important fields:

| Field | Purpose |
|------|---------|
| `running` | Starts/stops upload cycles |
| `mode` | `http_upload` or `udp` |
| `upload_url` | Receiver endpoint, for example `http://FOREIGN_IP:8080/upload` |
| `upload_token` | Shared token sent to the receiver |
| `ssh_host`, `ssh_user`, `ssh_password` | Used by Deploy Receiver |
| `receiver_port` | Remote receiver HTTP port |
| `min_cycle_upload_mb`, `max_cycle_upload_mb` | Upload size range |
| `cycle_sleep_seconds`, `max_cycle_sleep_seconds` | Sleep range between uploads |
| `schedule_enabled` | Enables time-based scheduling |

Keep `"running": false` until you intentionally start an authorized live test.

## Dashboard Workflow

1. Start `./hamgane`.
2. Login to the dashboard.
3. Fill **Target & Receiver**.
4. Click **Deploy Receiver** for TCP/HTTP mode.
5. Configure upload size, sleep range, randomizer, and schedule.
6. Click **Start Process**.
7. Watch live upload/download totals, speed, traffic balance, endpoint status, and upload progress.

## Commands

```bash
./hamgane
```

Starts the web dashboard.

```bash
./hamgane validate
```

Validates `.env` and `config.json`. It does not start upload cycles.

```bash
./hamgane receiver
```

Runs the HTTP receiver. This is normally started automatically on the foreign server by **Deploy Receiver**.

## Transfer Helper

From the source project root:

```bash
./transfer.sh
```

Option 1 packages `dist/hamgane`, `dist/config.json.example`, and `dist/.env.example` and uploads them to a server with `scp`.

Option 2 uploads a minimal GitHub bundle: `hamgane`, `config.json.example`, `.env.example`, and `README.md`.

## systemd Example

```ini
[Unit]
Description=HAMGANE web dashboard
After=network-online.target

[Service]
Restart=always
WorkingDirectory=/var/www/hamgane
ExecStart=/var/www/hamgane/hamgane

[Install]
WantedBy=multi-user.target
```

## Safety

- Use only in authorized labs, coursework, or owned infrastructure.
- Do not target third-party services or networks.
- Do not commit real IPs, SSH passwords, upload tokens, `.env`, or live `config.json`.
- Set `running=false` before sharing configs.
