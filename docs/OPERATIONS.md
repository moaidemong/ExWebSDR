# ExWebSDR Operations

## 24h Scheduler Mode

Run locally:

```bash
./run_title_stream.sh
```

This starts:
- `scheduler.py` (time-based band switching, A/B producer handover)
- `ws_server.py` (WebSocket gateway with metadata broadcast)
- `client` static server (`http://127.0.0.1:8079`)

## systemd Setup (WSL Ubuntu 22.04)

### 1) Create service file

Create `/etc/systemd/system/exwebsdr-title.service`:

```ini
[Unit]
Description=ExWebSDR 24h title stream
After=network.target

[Service]
Type=simple
User=kakut
WorkingDirectory=/mnt/c/Workspace/Codex/ExWebSDR
Environment=PYTHONUNBUFFERED=1
ExecStart=/usr/bin/env bash -lc 'source .venv/bin/activate && ./run_title_stream.sh'
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
```

Adjust `User` and `WorkingDirectory` as needed.

### 2) Reload and enable

```bash
sudo systemctl daemon-reload
sudo systemctl enable exwebsdr-title.service
sudo systemctl start exwebsdr-title.service
```

### 3) Verify and logs

```bash
sudo systemctl status exwebsdr-title.service
journalctl -u exwebsdr-title.service -f
```

## Notes

- Metadata overlay in browser can be toggled with `Meta On/Off` button or `M`.
- Band state file path default: `runtime/band_state.json`.

