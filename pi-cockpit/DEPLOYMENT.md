# Pi Cockpit — Deployment Guide

## Hardware

- Raspberry Pi 3 Model B/B+
- microSD with Raspberry Pi OS (Debian Bookworm, 64-bit)
- USB drive at `/media/peepo/KINGSTON` (or adjust paths)
- Phone on same WiFi network

---

## First-time Setup on Pi

### 1. Clone the repo

```bash
mkdir -p /media/peepo/KINGSTON
cd /media/peepo/KINGSTON
git clone https://github.com/HermannPR/lab6.git PI3
```

### 2. Install the systemd service

```bash
cd /media/peepo/KINGSTON/PI3/pi-cockpit
sudo cp setup/cockpit.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable cockpit.service
sudo systemctl start cockpit.service
```

### 3. Check status

```bash
sudo systemctl status cockpit.service
```

Open on phone: `http://<pi-ip>:5000`

---

## Updating

```bash
cd /media/peepo/KINGSTON/PI3
git fetch origin
git reset --hard origin/main
sudo systemctl restart cockpit.service
```

---

## Replacing an old `phone-server.service`

```bash
sudo systemctl disable phone-server.service
sudo systemctl stop phone-server.service
sudo systemctl enable cockpit.service
sudo systemctl start cockpit.service
```

---

## Architecture

```
projects/
  cockpit/
    server.py      unified Flask server — all routes in one process
    start.sh       dep install, QR code, exec sudo python3 server.py
    static/
      index.html   cockpit home UI (mode grid, health bar, power)

  phone-input/
    server.py      (legacy standalone — now served via cockpit)
    static/
      index.html   Claude phone UI — terminal stream, keyboard, mouse,
                   git dashboard, CLAUDE.md editor

  pegasus-pad/
    server.py      (legacy standalone — now served via cockpit)
    static/
      index.html   gamepad UI — D-pad, ABXY, L1/L2/R1/R2

setup/
  cockpit.service  systemd unit file
```

## URL map

| URL | What |
|-----|------|
| `http://<ip>:5000/` | Cockpit home — mode selector, health, power |
| `http://<ip>:5000/claude` | Phone input UI — Claude terminal |
| `http://<ip>:5000/gaming` | Pegasus gamepad |
| `http://<ip>:5000/ping` | Health check |
| `http://<ip>:5000/api/sys/info` | System stats JSON |
| `http://<ip>:5000/api/mode/status` | Current mode JSON |
| `http://<ip>:5000/api/mode/switch` | POST — switch mode |

## Boot modes

| Mode | What happens |
|------|------|
| `claude-dev` | Ensures tmux `setup:claude` session running Claude Code |
| `gaming` | Launches Pegasus frontend via `su peepo` |
| `desktop` | No process change — X11 desktop already running |
| `headless` | Kills Pegasus if running, no GUI |

Mode persists to `/tmp/pi-mode` (session) and `/etc/pi-mode` (boot default).

## Key endpoints

```
POST /api/mode/switch   { "mode": "gaming" }
POST /api/mode/default  { "mode": "claude-dev" }
POST /power/reboot
POST /power/shutdown
POST /prompt            { "text": "..." }   → sends to Claude tmux
POST /key               { "key": "Return" } → xdotool key
POST /mouse             { "btn": "click", "x": 100, "y": 200 }
GET  /claude-state      SSE stream — Claude terminal state
GET  /screen-stream     MJPEG screen capture
GET  /git/log           recent commits
GET  /git/show/<hash>   commit diff
POST /git/pull
GET  /file?path=...     read file (sandboxed to PI3 dir)
POST /file              write file
```

## Dependencies (auto-installed by start.sh)

```
flask flask-sock pillow python-uinput qrcode
xdotool (apt)
```

## Systemd service

```ini
[Unit]
Description=Pi Cockpit — unified phone control server
After=graphical.target media-peepo-KINGSTON.mount
Requires=media-peepo-KINGSTON.mount

[Service]
Type=simple
ExecStartPre=/bin/sleep 8
ExecStart=/bin/bash /media/peepo/KINGSTON/PI3/pi-cockpit/projects/cockpit/start.sh
Restart=on-failure
RestartSec=5

[Install]
WantedBy=graphical.target
```

The 8s pre-sleep ensures the USB drive mount is ready.
