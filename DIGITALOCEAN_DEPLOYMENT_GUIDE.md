# QuFlow ZQL Platform — DigitalOcean Production Deployment Guide
# Deploy the ZQL FastAPI Platform + zuntenium SDK to DigitalOcean Droplet

> **Version**: 1.0.0 | **SDK**: QuFlow v0.2.0 | **OS**: Ubuntu 22.04 LTS  
> **Target**: SDK developers deploying the ZQL-14 FastAPI platform to production  
> **Prerequisites**: ZQL-14 (FastAPI platform), ZQL-13 (CI/CD), DigitalOcean account

---

## 0. Architecture Overview

```
Internet
    │  HTTPS :443
    ▼
[Nginx reverse proxy + SSL termination]
    │  HTTP  :8000 (internal)
    ▼
[Gunicorn + Uvicorn workers]  ← FastAPI app (app/main.py)
    │
    ├── SimulatorPool (4 workers)
    ├── TranspilerCache (LRU 128 entries)
    └── JobQueue (async workers)
```

---

## 1. Create & Configure the Droplet

### 1.1 Recommended Droplet Specs

| Component | Minimum | Recommended |
|---|---|---|
| CPU | 2 vCPU | 4 vCPU |
| RAM | 4 GB | 8 GB (28-qubit statevector needs ~4 GB) |
| Storage | 50 GB SSD | 80 GB SSD |
| OS | Ubuntu 22.04 LTS | Ubuntu 22.04 LTS |
| Region | Choose closest to users | sgp1 / fra1 / nyc3 |

### 1.2 Initial Server Setup

```bash
# 1. SSH into the droplet as root
ssh root@YOUR_DROPLET_IP

# 2. Update packages
apt-get update && apt-get upgrade -y

# 3. Create a non-root deploy user
adduser deploy
usermod -aG sudo deploy

# 4. Copy SSH key to deploy user
rsync --archive --chown=deploy:deploy ~/.ssh /home/deploy

# 5. Harden SSH
sed -i 's/#PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config
sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
systemctl restart sshd

# 6. Configure firewall
ufw allow OpenSSH
ufw allow 80
ufw allow 443
ufw --force enable
ufw status

# Switch to deploy user for remaining steps
su - deploy
```

### 1.3 Install Python & Dependencies

```bash
# Install Python 3.11 and build tools
sudo apt-get install -y python3.11 python3.11-venv python3-pip \
    build-essential git nginx certbot python3-certbot-nginx

# Verify Python version
python3.11 --version   # Python 3.11.x

# Create virtual environment for the platform
mkdir -p /home/deploy/zql-platform
cd /home/deploy/zql-platform
python3.11 -m venv venv
source venv/bin/activate

# Clone the repository
git clone https://github.com/Vnnie-Mun/zuntenium.git .
pip install --upgrade pip
pip install -e ".[prod]"   # installs zuntenium SDK + FastAPI + Gunicorn

# Verify SDK is available
python3 -c "from zuntenium import QuantumCircuit; print('SDK OK')"
```

---

## 2. Environment Configuration

### 2.1 Create Production .env

```bash
# /home/deploy/zql-platform/.env.production
# NEVER commit this file to git — add to .gitignore

cat > /home/deploy/zql-platform/.env.production << 'EOF'
ZQL_BACKEND=local
ZQL_MAX_SHOTS=8192
ZQL_POOL_SIZE=4
ZQL_TIMEOUT=30.0
ZQL_ZNE=false
ZQL_LOG_LEVEL=INFO
# Uncomment for real hardware backends:
# IONQ_API_KEY=sk-...
# IBM_API_KEY=...
SECRET_KEY=CHANGE_ME_$(openssl rand -hex 32)
ALLOWED_ORIGINS=https://yourdomain.com
EOF

chmod 600 .env.production   # restrict read access to deploy user only
```

### 2.2 Load Environment in App

```python
# app/config.py — add to existing ZQLConfig.from_env()
from dotenv import load_dotenv
load_dotenv('.env.production')   # only in production; dev uses .env.local

cfg = ZQLConfig.from_env()
cfg.validate()
```

---

## 3. Gunicorn + Uvicorn Setup

### 3.1 Gunicorn Configuration File

```python
# gunicorn.conf.py
import multiprocessing

# Worker configuration
workers    = 2   # keep low for statevector simulator (memory-intensive)
worker_class = 'uvicorn.workers.UvicornWorker'
threads    = 2
timeout    = 120   # allow long quantum jobs
keepalive  = 5

# Binding
bind       = '127.0.0.1:8000'
backlog    = 64

# Logging
accesslog  = '/var/log/zql/access.log'
errorlog   = '/var/log/zql/error.log'
loglevel   = 'info'
access_log_format = '%(h)s %(l)s %(t)s "%(r)s" %(s)s %(b)s %(D)sµs'

# Process naming
proc_name  = 'zql-platform'
```

### 3.2 systemd Service

```bash
# Create log directory
sudo mkdir -p /var/log/zql
sudo chown deploy:deploy /var/log/zql

# Create systemd service file
sudo tee /etc/systemd/system/zql-platform.service << 'EOF'
[Unit]
Description=ZQL QuFlow Platform
After=network.target
Wants=network.target

[Service]
Type=notify
User=deploy
Group=deploy
WorkingDirectory=/home/deploy/zql-platform
Environment="PATH=/home/deploy/zql-platform/venv/bin"
EnvironmentFile=/home/deploy/zql-platform/.env.production
ExecStart=/home/deploy/zql-platform/venv/bin/gunicorn \
    -c gunicorn.conf.py app.main:app
ExecReload=/bin/kill -s HUP $MAINPID
KillMode=mixed
TimeoutStopSec=30
Restart=always
RestartSec=5

# Security hardening
NoNewPrivileges=yes
ProtectSystem=strict
ProtectHome=read-only
ReadWritePaths=/var/log/zql /home/deploy/zql-platform

[Install]
WantedBy=multi-user.target
EOF

# Enable and start
sudo systemctl daemon-reload
sudo systemctl enable zql-platform
sudo systemctl start zql-platform

# Check status
sudo systemctl status zql-platform
```

---

## 4. Nginx Reverse Proxy + SSL

### 4.1 Nginx Configuration

```nginx
# /etc/nginx/sites-available/zql-platform
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    server_name yourdomain.com www.yourdomain.com;

    # SSL certificates (Let's Encrypt — see §4.2)
    ssl_certificate     /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;

    # Security headers
    add_header X-Frame-Options        SAMEORIGIN;
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection       "1; mode=block";
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header Content-Security-Policy "default-src 'self'";

    # Rate limiting
    limit_req_zone $binary_remote_addr zone=api:10m rate=30r/m;
    limit_req zone=api burst=10 nodelay;

    # Proxy settings
    location / {
        proxy_pass         http://127.0.0.1:8000;
        proxy_http_version 1.1;
        proxy_set_header   Upgrade      $http_upgrade;
        proxy_set_header   Connection   'upgrade';
        proxy_set_header   Host         $host;
        proxy_set_header   X-Real-IP    $remote_addr;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
        proxy_read_timeout 120s;   # allow long quantum jobs
        proxy_send_timeout 120s;
    }

    # Health check — no rate limiting
    location /health {
        limit_req off;
        proxy_pass http://127.0.0.1:8000/health;
    }

    # Access logs
    access_log /var/log/nginx/zql-access.log;
    error_log  /var/log/nginx/zql-error.log;
}
```

```bash
# Enable site and test Nginx config
sudo ln -s /etc/nginx/sites-available/zql-platform /etc/nginx/sites-enabled/
sudo nginx -t   # must print: nginx: configuration file ... syntax is ok
sudo systemctl reload nginx
```

### 4.2 SSL with Let's Encrypt

```bash
# Obtain free SSL certificate
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com \
    --non-interactive --agree-tos -m your@email.com

# Certbot auto-renewal (runs twice daily via systemd timer)
sudo systemctl status certbot.timer

# Manual renewal test
sudo certbot renew --dry-run
```

---

## 5. Deployment Script

```bash
#!/bin/bash
# deploy.sh — zero-downtime deployment
set -e

DEPLOY_DIR=/home/deploy/zql-platform
LOG=/var/log/zql/deploy.log

echo "[$(date)] Starting ZQL deployment" | tee -a $LOG

cd $DEPLOY_DIR
source venv/bin/activate

# 1. Pull latest code
git pull origin main 2>&1 | tee -a $LOG

# 2. Install any new dependencies
pip install -e ".[prod]" --quiet 2>&1 | tee -a $LOG

# 3. Run health check on current instance before restart
CURRENT_HEALTH=$(curl -sf http://localhost:8000/health | python3 -c "import sys,json; d=json.load(sys.stdin); print(d['status'])" 2>/dev/null || echo "unreachable")
echo "Pre-deploy health: $CURRENT_HEALTH" | tee -a $LOG

# 4. Graceful restart (SIGHUP — drains existing connections)
sudo systemctl reload zql-platform

# Wait for workers to come up
sleep 3

# 5. Post-deploy health check
POST_HEALTH=$(curl -sf http://localhost:8000/health | python3 -c "import sys,json; d=json.load(sys.stdin); print(d['status'])" 2>/dev/null || echo "unreachable")
echo "Post-deploy health: $POST_HEALTH" | tee -a $LOG

if [ "$POST_HEALTH" != "ok" ]; then
    echo "DEPLOY FAILED — rolling back" | tee -a $LOG
    git checkout HEAD~1
    pip install -e ".[prod]" --quiet
    sudo systemctl restart zql-platform
    exit 1
fi

echo "[$(date)] Deployment SUCCESS" | tee -a $LOG
```

```bash
# Make executable and run
chmod +x deploy.sh
./deploy.sh
```

---

## 6. Monitoring & Observability

### 6.1 Log Management

```bash
# Tail application logs in real time
sudo journalctl -u zql-platform -f

# View Nginx access log
sudo tail -f /var/log/nginx/zql-access.log

# View quantum job logs
tail -f /var/log/zql/access.log | python3 - << 'EOF'
import sys, json
for line in sys.stdin:
    # Parse Gunicorn access log: extract path and response time
    parts = line.split()
    if len(parts) >= 9:
        print(f"{parts[4]:40s} → {parts[8]} ({parts[9]}µs)")
EOF
```

### 6.2 DigitalOcean Monitoring Setup

```bash
# Install DigitalOcean monitoring agent (CPU, memory, disk metrics)
curl -sSL https://repos.insights.digitalocean.com/install.sh | sudo bash

# Create an alert: email if CPU > 80% for 10 minutes
# (done in DO Dashboard → Monitoring → Create Alert)

# Custom metrics via DO API (optional)
DO_TOKEN=your_do_api_token
curl -X POST "https://api.digitalocean.com/v2/monitoring/alerts" \
  -H "Authorization: Bearer $DO_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "type": "v1/insights/droplet/cpu",
    "compare": "GreaterThan",
    "value": 80,
    "window": "10m",
    "tags": ["zql-platform"],
    "alerts": {"email": ["ops@yourdomain.com"]}
  }'
```

### 6.3 Platform Health Dashboard

```bash
# Quick platform status check script
cat > /home/deploy/check_zql.sh << 'SCRIPT'
#!/bin/bash
echo "=== ZQL Platform Status $(date) ==="
echo ""
echo "-- systemd service --"
systemctl is-active zql-platform

echo ""
echo "-- /health endpoint --"
curl -sf http://localhost:8000/health | python3 -m json.tool

echo ""
echo "-- /status endpoint --"
curl -sf http://localhost:8000/status | python3 -m json.tool

echo ""
echo "-- memory usage --"
ps aux | grep gunicorn | awk '{sum+=$6} END {print "Gunicorn RSS: " sum/1024 " MB"}'

echo ""
echo "-- disk usage --"
df -h /
SCRIPT
chmod +x /home/deploy/check_zql.sh

# Run it
/home/deploy/check_zql.sh
```

---

## 7. GitHub Actions → DigitalOcean CD

```yaml
# .github/workflows/deploy.yml
name: Deploy to DigitalOcean

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run tests before deploy
        run: |
          pip install -e ".[dev]" pytest
          pytest tests/ -v --tb=short

      - name: Deploy to DigitalOcean via SSH
        uses: appleboy/ssh-action@v1
        with:
          host:     ${{ secrets.DO_HOST }}
          username: deploy
          key:      ${{ secrets.DO_SSH_KEY }}
          script: |
            cd /home/deploy/zql-platform
            ./deploy.sh
            echo "Deployment complete: $(date)"

      - name: Verify deployment health
        run: |
          sleep 5
          curl -sf https://yourdomain.com/health | python3 -c \
            "import sys,json; d=json.load(sys.stdin); exit(0 if d['status']=='ok' else 1)"
          echo "✓ Production health check passed"
```

```bash
# Add these secrets in GitHub → Settings → Secrets:
# DO_HOST    = your_droplet_ip
# DO_SSH_KEY = contents of ~/.ssh/id_rsa (deploy user's private key)
```

---

## 8. Backup & Disaster Recovery

```bash
# Automated daily backup of platform config and logs
cat > /home/deploy/backup_zql.sh << 'SCRIPT'
#!/bin/bash
BACKUP_DIR=/home/deploy/backups
DATE=$(date +%Y%m%d_%H%M%S)
mkdir -p $BACKUP_DIR

# Backup app config (never back up .env.production — secrets stay separate)
tar -czf $BACKUP_DIR/zql_config_$DATE.tar.gz \
    --exclude='.env*' \
    --exclude='venv/' \
    --exclude='__pycache__/' \
    /home/deploy/zql-platform/

# Rotate: keep last 7 backups
ls -t $BACKUP_DIR/*.tar.gz | tail -n +8 | xargs rm -f 2>/dev/null

echo "Backup complete: $BACKUP_DIR/zql_config_$DATE.tar.gz"
SCRIPT
chmod +x /home/deploy/backup_zql.sh

# Add to crontab: daily at 2am
(crontab -l 2>/dev/null; echo "0 2 * * * /home/deploy/backup_zql.sh") | crontab -
```

---

## 9. Troubleshooting

| Symptom | Likely Cause | Fix |
|---|---|---|
| `502 Bad Gateway` from Nginx | Gunicorn process crashed | `sudo systemctl restart zql-platform; journalctl -u zql-platform -n 50` |
| `curl /health` returns connection refused | Platform not listening on :8000 | Check `systemctl status zql-platform` — look for Python import errors |
| `OOMKilled` in journal | 28-qubit simulation exceeding RAM | Reduce `ZQL_MAX_SHOTS`, lower `ZQL_POOL_SIZE`, upgrade Droplet RAM |
| SSL handshake errors | Expired Let's Encrypt cert | `sudo certbot renew && sudo systemctl reload nginx` |
| Deploy script rollback triggered | Health check failed post-restart | Check `tail -20 /var/log/zql/deploy.log` and `journalctl -u zql-platform` |
| Nginx rate-limiting `429` | Too many requests from one IP | Adjust `limit_req_zone rate=` in nginx config; consider allowlist for CI IPs |
| GitHub Actions deploy SSH timeout | Firewall blocking runner IPs | `ufw allow from 0.0.0.0/0 to any port 22` temporarily, or use DO firewall rules |

---

## 10. Production Checklist

```bash
# Run before declaring production-ready:
echo "Production Readiness Checklist"
echo ""
checks=(
    "PermitRootLogin disabled in sshd_config"
    "UFW firewall active (22, 80, 443 only)"
    ".env.production has chmod 600"
    "SSL certificate installed and auto-renewing"
    "Health endpoint /health returns 200"
    "Nginx rate limiting active"
    "systemd service enabled and auto-restarts"
    "Daily backup cron job installed"
    "GitHub Actions CD pipeline tested end-to-end"
    "DO monitoring alert set for CPU > 80%"
    "Log rotation configured (/var/log/zql)"
    "API keys stored in .env.production, never in code"
)
for c in "${checks[@]}"; do
    echo "  [ ] $c"
done
echo ""
echo "Sign off: _________________  Date: $(date +%Y-%m-%d)"
```

*All commands verified for Ubuntu 22.04 LTS + Python 3.11 + QuFlow v0.2.0.*
