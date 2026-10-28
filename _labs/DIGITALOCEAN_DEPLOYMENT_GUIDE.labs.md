# DigitalOcean Deployment Guide — Walkthrough Labs

**Source:** [DIGITALOCEAN_DEPLOYMENT_GUIDE.md](../DIGITALOCEAN_DEPLOYMENT_GUIDE.md)
**Generated:** 2026-10-28
**Labs:** 20 | **Domain:** Platform Engineering (build-and-ship)

---

## Source Analysis

- **Document objective:** Guide SDK developers through deploying the ZQL FastAPI quantum-computing platform to a DigitalOcean Droplet with Nginx, SSL, CI/CD, monitoring, and backup.
- **Target audience:** Intermediate developers familiar with Python and Linux; prerequisites are ZQL-13 (CI/CD) and ZQL-14 (FastAPI platform).
- **Domain classification:** Platform engineering / DevOps deployment.

### Concept Inventory (27 items)

1. Cloud droplet provisioning and sizing
2. SSH hardening (key-only, no root)
3. UFW firewall configuration
4. Python virtual-environment isolation
5. Environment-variable-driven configuration (.env)
6. Gunicorn worker model (process vs. thread)
7. Uvicorn ASGI worker class
8. systemd service management
9. systemd security hardening directives
10. Nginx reverse-proxy fundamentals
11. HTTP→HTTPS redirect
12. SSL/TLS with Let's Encrypt + Certbot
13. Security headers (HSTS, CSP, X-Frame-Options)
14. Nginx rate limiting (leaky bucket)
15. Health-check endpoints
16. Zero-downtime deployment (graceful reload)
17. Deployment-script rollback logic
18. GitHub Actions CD pipeline
19. SSH-based remote deployment
20. Log management (journalctl, access logs)
21. DigitalOcean monitoring agent & alerts
22. Custom status-check scripts
23. Backup & rotation with cron
24. Disaster recovery (restore from backup)
25. Production-readiness checklist
26. Memory-aware capacity planning (statevector RAM)
27. CORS / allowed-origins configuration

### Selected 20 Concepts (difficulty ramp)

| # | Concept | Approach |
|---|---------|----------|
| 1 | Cloud droplet provisioning | Provision and verify a minimal Ubuntu server |
| 2 | SSH hardening | Harden SSH config and test lockout |
| 3 | UFW firewall | Build a firewall rule set from scratch |
| 4 | Python virtualenv isolation | Create isolated env and prove isolation |
| 5 | Environment variable config | Design a .env-driven config loader |
| 6 | Gunicorn worker model | Benchmark workers × threads combos |
| 7 | systemd service management | Write, enable, and lifecycle-test a unit |
| 8 | Nginx reverse proxy | Configure proxy_pass and verify headers |
| 9 | SSL with Let's Encrypt | Obtain, verify, and auto-renew certs |
| 10 | Security headers | Implement and audit response headers |
| 11 | Nginx rate limiting | Configure and load-test rate limits |
| 12 | Health-check endpoints | Design health/status API contracts |
| 13 | Zero-downtime deploy script | Build and test graceful-reload deploy |
| 14 | Deployment rollback logic | Inject failure and verify auto-rollback |
| 15 | GitHub Actions CD pipeline | Build full CI→CD→health-verify workflow |
| 16 | Log management & parsing | Aggregate, parse, and alert on logs |
| 17 | Monitoring & alerting | Set up DO monitoring + custom checks |
| 18 | Backup & disaster recovery | Automate backup, rotation, and restore |
| 19 | Capacity planning (Africa context) | Right-size for low-budget, mobile-first |
| 20 | Production readiness audit | Comprehensive security & ops audit |

---

## How to Use This Lab Pack

**Environment:** Ubuntu 22.04 LTS (real Droplet, Multipass VM, or Docker container). Python 3.11+. Git, curl, nginx, certbot installed.

**Tooling:** Terminal, SSH client, text editor (nano/vim/VS Code Remote-SSH).

**Time estimate:** ~30–40 hours total (1.5–2 hours per lab).

**Progression:** Labs build sequentially. Labs 1–5 set up the server, 6–12 configure the application stack, 13–18 cover deployment and operations, 19–20 are capstone-level.

---

## Global Setup

```bash
# Option A: Real DigitalOcean Droplet (recommended for Labs 9, 15, 17)
# Create via DO Dashboard: Ubuntu 22.04, 4 GB RAM, 2 vCPU

# Option B: Local VM with Multipass (free, works offline)
# Install Multipass: https://multipass.run
multipass launch 22.04 --name zql-lab --cpus 2 --memory 4G --disk 20G
multipass shell zql-lab

# On the VM/droplet:
sudo apt-get update && sudo apt-get upgrade -y
sudo apt-get install -y python3.11 python3.11-venv python3-pip \
    build-essential git nginx curl jq

# Create workspace
mkdir -p ~/zql-labs && cd ~/zql-labs
python3.11 -m venv venv && source venv/bin/activate
pip install --upgrade pip

# Smoke test
python3 -c "import sys; assert sys.version_info >= (3,11); print('Python OK')"
nginx -v
echo "Global setup complete"
```

---

## Lab Map

| # | Title | Concept | Approach | Difficulty | Est. Time | Prerequisites |
|---|-------|---------|----------|------------|-----------|---------------|
| 1 | Provision Your First Cloud Server | Droplet provisioning | CLI-driven setup | Beginner | 60 min | None |
| 2 | Lock the Door: SSH Hardening | SSH security | Harden & test lockout | Beginner | 60 min | Lab 1 |
| 3 | Build a Firewall from Scratch | UFW firewall | Rule-by-rule build | Beginner | 60 min | Lab 2 |
| 4 | Isolate with Virtual Environments | Virtualenv isolation | Prove dependency isolation | Beginner | 45 min | Lab 1 |
| 5 | Secrets Management with .env | Env-var config | Build config loader | Beginner | 75 min | Lab 4 |
| 6 | Tune Your ASGI Engine | Gunicorn workers | Benchmark worker configs | Intermediate | 90 min | Lab 5 |
| 7 | systemd: Your App as a Service | systemd units | Full lifecycle test | Intermediate | 90 min | Lab 6 |
| 8 | Reverse Proxy with Nginx | Nginx proxy | Header verification | Intermediate | 90 min | Lab 7 |
| 9 | HTTPS with Let's Encrypt | SSL/TLS | Cert lifecycle | Intermediate | 75 min | Lab 8 |
| 10 | Armor Your Headers | Security headers | Audit & fix | Intermediate | 60 min | Lab 9 |
| 11 | Rate Limiting Under Load | Nginx rate limiting | Load-test & tune | Intermediate | 90 min | Lab 8 |
| 12 | Design a Health-Check API | Health endpoints | Contract-first design | Intermediate | 75 min | Lab 7 |
| 13 | Zero-Downtime Deploys | Graceful reload | Build deploy script | Advanced | 90 min | Lab 8 |
| 14 | When Deploys Fail: Rollback | Rollback logic | Inject failures | Advanced | 90 min | Lab 13 |
| 15 | CI/CD with GitHub Actions | CD pipeline | End-to-end workflow | Advanced | 120 min | Lab 14 |
| 16 | Log Forensics | Log management | Parse & alert | Advanced | 90 min | Lab 7 |
| 17 | Monitoring & Alerting | DO monitoring | Custom dashboards | Advanced | 90 min | Lab 16 |
| 18 | Backup & Disaster Recovery | Backup/restore | Automate & test restore | Advanced | 90 min | Lab 7 |
| 19 | Right-Size for Africa | Capacity planning | Low-budget optimization | Expert | 120 min | Labs 6,17 |
| 20 | Production Readiness Audit | Security audit | Full checklist audit | Expert | 120 min | All |

---

## Lab 1: Provision Your First Cloud Server

- **Concept:** Cloud droplet provisioning and initial server sizing
- **Approach:** Use the DigitalOcean CLI (`doctl`) to provision, configure, and verify a Droplet entirely from the command line
- **Difficulty:** Beginner | **Time:** 60 min | **Prerequisites:** None
- **Learning outcomes:**
  1. You will be able to provision a DigitalOcean Droplet using the CLI
  2. You will be able to select appropriate CPU/RAM/storage for quantum workloads
  3. You will be able to verify SSH connectivity to a new server
  4. You will be able to explain why statevector simulation drives memory requirements
- **Why this matters:** Every Zuntenium QuantumHub deployment starts with a properly sized server. Under-provisioning causes OOMKilled crashes during 28-qubit simulations; over-provisioning wastes budget — critical for African institutions with tight compute budgets.

### Theory Primer

A DigitalOcean Droplet is a virtual private server. The ZQL platform runs statevector simulation, which stores the full quantum state as a 2^n complex vector. For n=28 qubits:

```
Memory = 2^28 × 16 bytes (complex128) = 4,294,967,296 bytes ≈ 4 GB
```

The deployment guide recommends 8 GB RAM to leave headroom for the OS, Gunicorn workers, and Nginx. The `doctl` CLI lets you script provisioning, making it repeatable and auditable.

*(Ref: Source doc §0 Architecture Overview, §1.1 Recommended Droplet Specs)*

### Setup

```bash
# Install doctl (DigitalOcean CLI)
# macOS:
brew install doctl
# Linux:
sudo snap install doctl

# Authenticate
doctl auth init   # paste your DO API token when prompted

# Verify
doctl account get
```

### Walkthrough

**Step 1: List available Droplet sizes and pick a size slug.**

```bash
doctl compute size list --format Slug,Memory,VCPUs,Disk,PriceMonthly | head -20
```

What it does: Queries the DO API for all available Droplet sizes.

Expected output (excerpt):
```
Slug               Memory    VCPUs    Disk    PriceMonthly
s-1vcpu-1gb        1024      1        25      6.00
s-2vcpu-4gb        4096      2        80      24.00
s-4vcpu-8gb        8192      4        160     48.00
```

**Checkpoint:** You should see a table of sizes. Identify `s-2vcpu-4gb` (minimum) and `s-4vcpu-8gb` (recommended).

**Step 2: List available regions.**

```bash
doctl compute region list --format Slug,Name,Available | grep true
```

What it does: Shows regions where Droplets can be created. For African deployments, `fra1` (Frankfurt) or `lon1` (London) gives lowest latency to East Africa.

**Step 3: Create an SSH key on your local machine (if you don't have one).**

```bash
ssh-keygen -t ed25519 -C "zql-lab" -f ~/.ssh/zql_lab_key -N ""
# Upload to DO
doctl compute ssh-key import zql-lab-key --public-key-file ~/.ssh/zql_lab_key.pub
# Note the key ID
doctl compute ssh-key list --format ID,Name
```

What it does: Creates an Ed25519 keypair and registers the public key with DigitalOcean so the Droplet can be created with key-based access.

**Step 4: Provision the Droplet.**

```bash
doctl compute droplet create zql-lab-01 \
    --size s-2vcpu-4gb \
    --image ubuntu-22-04-x64 \
    --region fra1 \
    --ssh-keys $(doctl compute ssh-key list --format ID --no-header | head -1) \
    --tag-name zql-platform \
    --wait
```

What it does: Creates a 4 GB / 2 vCPU Droplet running Ubuntu 22.04 in Frankfurt with your SSH key pre-installed. `--wait` blocks until the Droplet is active.

Expected output:
```
ID           Name          Public IPv4       Status
123456789    zql-lab-01    164.90.xxx.xxx    active
```

**Checkpoint:** You should see status `active` and a public IP address.

**Step 5: SSH into the new Droplet.**

```bash
export ZQL_IP=$(doctl compute droplet get zql-lab-01 --format PublicIPv4 --no-header)
ssh -i ~/.ssh/zql_lab_key root@$ZQL_IP "uname -a && free -h && df -h /"
```

Expected output:
```
Linux zql-lab-01 5.15.0-xx-generic ... x86_64 GNU/Linux
               total        used        free
Mem:           3.8Gi       200Mi       3.4Gi
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1        78G  2.1G   72G   3% /
```

**Checkpoint:** Kernel is 5.15+, ~3.8 GB RAM visible, ~72 GB disk free.

**Step 6: Record server details for later labs.**

```bash
cat > ~/zql-labs/server_info.env << EOF
ZQL_DROPLET_IP=$ZQL_IP
ZQL_DROPLET_ID=$(doctl compute droplet get zql-lab-01 --format ID --no-header)
ZQL_REGION=fra1
ZQL_SIZE=s-2vcpu-4gb
EOF
cat ~/zql-labs/server_info.env
```

### Verification

```bash
# Assert Droplet is active
STATUS=$(doctl compute droplet get zql-lab-01 --format Status --no-header)
[ "$STATUS" = "active" ] && echo "PASS: Droplet active" || echo "FAIL: status=$STATUS"

# Assert SSH works
ssh -i ~/.ssh/zql_lab_key -o ConnectTimeout=5 root@$ZQL_IP "echo PASS" 2>/dev/null || echo "FAIL: SSH"

# Assert RAM >= 3500 MB
RAM_MB=$(ssh -i ~/.ssh/zql_lab_key root@$ZQL_IP "free -m | awk '/Mem:/{print \$2}'")
[ "$RAM_MB" -ge 3500 ] && echo "PASS: RAM=${RAM_MB}MB" || echo "FAIL: RAM=${RAM_MB}MB"
```

### Break It on Purpose

**Deliberately create a Droplet that is too small:**

```bash
doctl compute droplet create zql-tiny \
    --size s-1vcpu-1gb --image ubuntu-22-04-x64 --region fra1 \
    --ssh-keys $(doctl compute ssh-key list --format ID --no-header | head -1) --wait

TINY_IP=$(doctl compute droplet get zql-tiny --format PublicIPv4 --no-header)
ssh root@$TINY_IP "python3 -c \"
import numpy as np
# Attempt 28-qubit statevector — needs 4 GB
v = np.zeros(2**28, dtype=np.complex128)
print('Allocated', v.nbytes / 1e9, 'GB')
\""
```

**Diagnosis:** The process will be OOMKilled. Check with `dmesg | tail -5`. This proves why the deployment guide specifies 4–8 GB minimum.

**Cleanup:** `doctl compute droplet delete zql-tiny --force`

### Compare Approaches

| Approach | Pros | Cons | When to use |
|----------|------|------|-------------|
| `doctl` CLI | Scriptable, repeatable | Requires API token setup | Automated provisioning, CI/CD |
| DO Dashboard (Web UI) | Visual, beginner-friendly | Not scriptable, click-drift | One-off exploration |
| Terraform | Full IaC, state tracking | Steeper learning curve | Multi-resource infra |

### Stretch Challenges

1. **Easy:** Create a Droplet in a different region and measure SSH latency from your location using `ping` and `ssh -o ProxyCommand` timing.
2. **Medium:** Write a bash script that creates a Droplet, waits for it, runs a smoke test, and tears it down — all in one command.
3. **Hard:** Use the DO API directly (curl + JSON) instead of `doctl` to create and verify a Droplet.

<details><summary>Hints & Solutions</summary>

**Challenge 3 hint:** `curl -X POST "https://api.digitalocean.com/v2/droplets" -H "Authorization: Bearer $DO_TOKEN" -H "Content-Type: application/json" -d '{"name":"zql-api-test","region":"fra1","size":"s-2vcpu-4gb","image":"ubuntu-22-04-x64","ssh_keys":["YOUR_KEY_FINGERPRINT"]}'`

</details>

### Common Pitfalls

| Mistake | Symptom | Fix |
|---------|---------|-----|
| Forgetting `--wait` | Script proceeds before Droplet has an IP | Add `--wait` flag |
| Wrong SSH key | `Permission denied (publickey)` | Verify key ID matches: `doctl compute ssh-key list` |
| Choosing a region without capacity | Creation fails | Pick a different region slug |

### Reflection Questions

1. Why does a 28-qubit statevector need exactly 4 GB of RAM? Walk through the calculation.
2. If you were deploying QuantumHub for a university in Nairobi, which DO region would minimize latency and why?
3. What is the cost difference between the minimum (4 GB) and recommended (8 GB) Droplet over one year? When is the upgrade justified?

### Deliverable

`~/zql-labs/server_info.env` file with your Droplet's IP, ID, region, and size — ready for all subsequent labs.

---

## Lab 2: Lock the Door — SSH Hardening

- **Concept:** SSH hardening (disable root login, disable password auth, key-only access)
- **Approach:** Manually harden sshd_config, verify lockout, and test that only authorized keys work
- **Difficulty:** Beginner | **Time:** 60 min | **Prerequisites:** Lab 1
- **Learning outcomes:**
  1. You will be able to create a non-root deploy user with sudo access
  2. You will be able to disable root login and password authentication in sshd_config
  3. You will be able to verify that unauthorized access attempts are rejected
  4. You will be able to audit SSH login attempts from system logs
- **Why this matters:** An internet-facing server running a quantum API is a high-value target. The deployment guide mandates key-only SSH as the first security layer. Brute-force bots start probing within minutes of a Droplet going live.

### Theory Primer

SSH (Secure Shell) authenticates via passwords or cryptographic keys. Password auth is vulnerable to brute force. Key auth uses public-key cryptography: the server stores your public key, and only the holder of the matching private key can authenticate.

```
Client (private key) ──── challenge/response ────► Server (authorized_keys)
        ✓ match → access granted
        ✗ no match → access denied
```

sshd_config directives (from source doc §1.2):
- `PermitRootLogin no` — blocks direct root SSH
- `PasswordAuthentication no` — forces key-only auth

*(Ref: Source doc §1.2 Initial Server Setup)*

### Setup

```bash
# SSH into your Droplet from Lab 1
source ~/zql-labs/server_info.env
ssh -i ~/.ssh/zql_lab_key root@$ZQL_DROPLET_IP
```

### Walkthrough

**Step 1: Create the deploy user.**

```bash
adduser deploy --gecos "" --disabled-password
echo "deploy:$(openssl rand -base64 32)" | chpasswd
usermod -aG sudo deploy
```

What it does: Creates user `deploy` with a random password (we'll disable password auth entirely), adds to the sudo group.

Expected output: No errors.

**Step 2: Copy SSH keys to the deploy user.**

```bash
rsync --archive --chown=deploy:deploy ~/.ssh /home/deploy
ls -la /home/deploy/.ssh/
```

Expected output:
```
-rw------- 1 deploy deploy  ... authorized_keys
```

**Checkpoint:** `authorized_keys` exists and is owned by `deploy`.

**Step 3: Test SSH as deploy (from a new local terminal).**

```bash
ssh -i ~/.ssh/zql_lab_key deploy@$ZQL_DROPLET_IP "whoami && sudo whoami"
```

Expected output:
```
deploy
root
```

**Checkpoint:** You can SSH as deploy and sudo works.

**Step 4: Harden sshd_config.**

```bash
# Back on the server as root:
cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak   # backup first!

sed -i 's/#PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config
sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin no/' /etc/ssh/sshd_config
sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
sed -i 's/PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config

# Verify the changes
grep -E "^(PermitRootLogin|PasswordAuthentication)" /etc/ssh/sshd_config
```

Expected output:
```
PermitRootLogin no
PasswordAuthentication no
```

**Step 5: Restart sshd.**

```bash
systemctl restart sshd
echo "sshd restarted"
```

**Step 6: Verify root login is blocked (from your local machine).**

```bash
ssh -i ~/.ssh/zql_lab_key root@$ZQL_DROPLET_IP "echo should not see this" 2>&1
```

Expected output:
```
root@...: Permission denied (publickey).
```

**Checkpoint:** Root login fails. This is the desired behavior.

### Verification

```bash
# Test 1: deploy user works
ssh -i ~/.ssh/zql_lab_key deploy@$ZQL_DROPLET_IP "echo PASS: deploy login"

# Test 2: root is blocked
ssh -i ~/.ssh/zql_lab_key root@$ZQL_DROPLET_IP "echo FAIL" 2>/dev/null
[ $? -ne 0 ] && echo "PASS: root blocked" || echo "FAIL: root still allowed"

# Test 3: password auth is disabled (will timeout/fail)
ssh -o PasswordAuthentication=yes -o PubkeyAuthentication=no \
    deploy@$ZQL_DROPLET_IP "echo FAIL" 2>/dev/null
[ $? -ne 0 ] && echo "PASS: password auth disabled" || echo "FAIL: password auth works"
```

### Break It on Purpose

**Remove the deploy user's authorized_keys and try to log in:**

```bash
# As deploy on the server:
mv ~/.ssh/authorized_keys ~/.ssh/authorized_keys.bak
# Now from local: try to SSH — it will fail
# Fix: restore from backup
mv ~/.ssh/authorized_keys.bak ~/.ssh/authorized_keys
```

**Diagnosis:** Without any authorized key, key auth fails. Without password auth, there's no fallback. This is why you should always keep a DO console session open when making SSH changes.

### Compare Approaches

| Approach | Security | Convenience | When to use |
|----------|----------|-------------|-------------|
| Key-only SSH | High | Medium (key management) | Production servers |
| Password + fail2ban | Medium | Easy | Dev/staging |
| Certificate-based SSH | Very high | Complex setup | Enterprise / fleet |

### Stretch Challenges

1. Install `fail2ban` and configure it to ban IPs after 3 failed SSH attempts in 10 minutes.
2. Change the SSH port from 22 to 2222 and update the UFW rules accordingly.
3. Set up SSH certificate authority (CA) so new team members get access without editing authorized_keys.

<details><summary>Hints</summary>

**Challenge 1:** `sudo apt install fail2ban && sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local` then set `maxretry = 3` under `[sshd]`.

</details>

### Common Pitfalls

| Mistake | Symptom | Fix |
|---------|---------|-----|
| Hardening SSH before copying keys to deploy | Locked out of server | Use DO console to recover |
| Editing sshd_config without backup | Can't undo if syntax error | Always `cp sshd_config sshd_config.bak` first |
| Not testing deploy SSH before disabling root | Both access paths fail | Test in parallel terminals |

### Reflection Questions

1. Why does the deployment guide disable root login rather than just using a strong root password?
2. What is the threat model that key-based auth protects against that passwords don't?
3. If a team member leaves, what steps must you take to revoke their SSH access?

### Deliverable

Hardened server with root login disabled, password auth disabled, and verified deploy-user key-based access. Screenshot or terminal log of the three verification tests passing.

---

## Lab 3: Build a Firewall from Scratch

- **Concept:** UFW (Uncomplicated Firewall) configuration
- **Approach:** Build firewall rules one at a time, testing connectivity after each rule, and verifying that blocked ports are truly unreachable
- **Difficulty:** Beginner | **Time:** 60 min | **Prerequisites:** Lab 2
- **Learning outcomes:**
  1. You will be able to configure UFW rules for specific ports and protocols
  2. You will be able to verify that blocked ports reject connections
  3. You will be able to explain the difference between allow, deny, and limit rules
  4. You will be able to audit active firewall rules
- **Why this matters:** The ZQL platform exposes HTTP/HTTPS and SSH. Every other port must be blocked. A misconfigured firewall can expose internal services (like Gunicorn on :8000) directly to the internet, bypassing Nginx's security headers and rate limiting.

### Theory Primer

UFW is a frontend for Linux's `iptables`/`nftables`. It processes rules in order — first match wins. The deployment guide (§1.2) opens only three ports:

```
Port 22  (SSH)   — remote administration
Port 80  (HTTP)  — redirect to HTTPS
Port 443 (HTTPS) — application traffic
```

Everything else is denied by default (`ufw default deny incoming`).

*(Ref: Source doc §1.2, firewall commands)*

### Setup

```bash
source ~/zql-labs/server_info.env
ssh -i ~/.ssh/zql_lab_key deploy@$ZQL_DROPLET_IP
```

### Walkthrough

**Step 1: Check current UFW status.**

```bash
sudo ufw status verbose
```

Expected output: `Status: inactive` (if this is a fresh server).

**Step 2: Set default policies.**

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

What it does: Blocks all inbound traffic by default, allows the server to make outbound connections (for apt, git, etc.).

**Step 3: Allow SSH first (critical — do this before enabling UFW).**

```bash
sudo ufw allow OpenSSH
# Or equivalently: sudo ufw allow 22/tcp
```

What it does: Opens port 22 so you don't lock yourself out.

**Step 4: Enable the firewall.**

```bash
sudo ufw --force enable
sudo ufw status numbered
```

Expected output:
```
Status: active

     To                         Action      From
     --                         ------      ----
[ 1] 22/tcp (OpenSSH)           ALLOW IN    Anywhere
```

**Checkpoint:** SSH still works (you're still connected). Only port 22 is open.

**Step 5: Start a test HTTP server on port 8080 and verify it's blocked.**

```bash
# On the server: start a simple HTTP server
python3 -m http.server 8080 &
HTTP_PID=$!

# From your local machine (new terminal):
curl -sf --connect-timeout 3 http://$ZQL_DROPLET_IP:8080/ && echo "FAIL: port 8080 reachable" || echo "PASS: port 8080 blocked"

# Clean up
kill $HTTP_PID
```

Expected: `PASS: port 8080 blocked` — the firewall drops the connection.

**Step 6: Allow HTTP and HTTPS.**

```bash
sudo ufw allow 80/tcp comment 'HTTP redirect'
sudo ufw allow 443/tcp comment 'HTTPS'
sudo ufw status numbered
```

Expected output shows three rules (22, 80, 443).

**Step 7: Verify port 80 is now open.**

```bash
# On server: temporary listener
python3 -m http.server 80 &
HTTP_PID=$!

# From local:
curl -sf --connect-timeout 3 http://$ZQL_DROPLET_IP/ && echo "PASS: port 80 reachable"

# Clean up
sudo kill $HTTP_PID
```

### Verification

```bash
# Verify exactly 3 rules (SSH, HTTP, HTTPS)
RULE_COUNT=$(sudo ufw status numbered | grep -c "ALLOW IN")
[ "$RULE_COUNT" -eq 3 ] && echo "PASS: 3 rules active" || echo "FAIL: $RULE_COUNT rules"

# Verify internal port 8000 is NOT reachable from outside
python3 -m http.server 8000 &
PID=$!
sleep 1
# From local:
curl -sf --connect-timeout 3 http://$ZQL_DROPLET_IP:8000/ 2>/dev/null
[ $? -ne 0 ] && echo "PASS: 8000 blocked externally" || echo "FAIL: 8000 exposed!"
kill $PID
```

### Break It on Purpose

**Open port 8000 directly and see why this is dangerous:**

```bash
sudo ufw allow 8000/tcp
# Now Gunicorn would be directly accessible, bypassing Nginx
# No SSL, no rate limiting, no security headers
sudo ufw delete allow 8000/tcp   # fix immediately
```

### Compare Approaches

| Approach | Pros | Cons | When to use |
|----------|------|------|-------------|
| UFW | Simple syntax, beginner-friendly | Limited advanced features | Single-server deploys |
| iptables/nftables | Full control, powerful | Complex syntax | Custom networking needs |
| DO Cloud Firewall | Network-level, no server config | Costs extra, less granular | Multi-droplet fleets |

### Stretch Challenges

1. Add a `limit` rule for SSH that auto-throttles IPs with >6 connections in 30 seconds.
2. Allow SSH only from your specific IP address (`sudo ufw allow from YOUR_IP to any port 22`).
3. Write a script that verifies all expected ports are open and all others are closed using `nmap`.

<details><summary>Hints</summary>

**Challenge 1:** `sudo ufw delete allow OpenSSH && sudo ufw limit OpenSSH`

**Challenge 3:** `nmap -p 1-1024 $ZQL_DROPLET_IP | grep -E "open|closed|filtered"`
</details>

### Common Pitfalls

| Mistake | Symptom | Fix |
|---------|---------|-----|
| Enabling UFW before allowing SSH | Locked out | Use DO console; `ufw allow 22` |
| Forgetting `--force` flag | UFW prompts and script hangs | Always use `--force` in scripts |
| Opening port 8000 | Gunicorn exposed directly | Only open 80/443; keep 8000 internal |

### Reflection Questions

1. Why does the deployment guide use `127.0.0.1:8000` for Gunicorn binding instead of `0.0.0.0:8000`?
2. What happens if you accidentally delete the SSH rule while connected? Do you lose your session?
3. How would you configure UFW differently for a server behind a load balancer?

### Deliverable

Server with UFW active, exactly 3 rules (22, 80, 443), and a terminal log showing that port 8000 is blocked externally. Save the output of `sudo ufw status verbose` to `~/zql-labs/lab03_firewall.txt`.

---

## Lab 4: Isolate with Virtual Environments

- **Concept:** Python virtual-environment isolation for production deployments
- **Approach:** Create multiple venvs, prove they are isolated, and demonstrate what happens when you skip isolation
- **Difficulty:** Beginner | **Time:** 45 min | **Prerequisites:** Lab 1
- **Learning outcomes:**
  1. You will be able to create and activate Python virtual environments
  2. You will be able to prove that packages in one venv are invisible to another
  3. You will be able to explain why venvs are critical for production deployments
  4. You will be able to diagnose "ModuleNotFoundError" caused by wrong venv activation
- **Why this matters:** The ZQL platform (source doc §1.3) uses a venv to pin its exact dependency set. Without isolation, system Python updates or other applications can break the quantum SDK silently.

### Theory Primer

A Python virtual environment is a self-contained directory tree containing a Python interpreter and a private site-packages. When activated, `pip install` targets the venv, not the system Python.

```
System Python 3.11
├── /usr/lib/python3.11/site-packages/   ← system packages
│
├── venv-A/lib/python3.11/site-packages/ ← venv A packages
│   └── numpy==1.26.4
│
└── venv-B/lib/python3.11/site-packages/ ← venv B packages
    └── numpy==2.0.0   ← different version, no conflict
```

*(Ref: Source doc §1.3 — `python3.11 -m venv venv`)*

### Setup

```bash
ssh -i ~/.ssh/zql_lab_key deploy@$ZQL_DROPLET_IP
cd ~/zql-labs
```

### Walkthrough

**Step 1: Create two separate virtual environments.**

```bash
python3.11 -m venv venv-prod
python3.11 -m venv venv-test
ls -d venv-*/
```

Expected output: `venv-prod/  venv-test/`

**Step 2: Install different versions of numpy in each venv.**

```bash
# In venv-prod
source venv-prod/bin/activate
pip install numpy==1.26.4 --quiet
python3 -c "import numpy; print(f'prod: numpy {numpy.__version__}')"
deactivate

# In venv-test
source venv-test/bin/activate
pip install numpy==2.0.0 --quiet
python3 -c "import numpy; print(f'test: numpy {numpy.__version__}')"
deactivate
```

Expected output:
```
prod: numpy 1.26.4
test: numpy 2.0.0
```

**Checkpoint:** Two different numpy versions coexist without conflict.

**Step 3: Prove isolation — venv-prod cannot see venv-test's packages.**

```bash
source venv-prod/bin/activate
pip install scipy==1.13.0 --quiet   # only in prod
deactivate

source venv-test/bin/activate
python3 -c "import scipy" 2>&1  # should fail
deactivate
```

Expected output (from venv-test):
```
ModuleNotFoundError: No module named 'scipy'
```

**Step 4: Create the production-style venv as specified in the deployment guide.**

```bash
python3.11 -m venv venv
source venv/bin/activate
pip install --upgrade pip
pip install fastapi uvicorn gunicorn python-dotenv --quiet
python3 -c "import fastapi; print(f'FastAPI {fastapi.__version__}')"
```

**Checkpoint:** FastAPI imports successfully in the production venv.

### Verification

```bash
source venv-prod/bin/activate
PROD_NP=$(python3 -c "import numpy; print(numpy.__version__)")
deactivate
source venv-test/bin/activate
TEST_NP=$(python3 -c "import numpy; print(numpy.__version__)")
deactivate

[ "$PROD_NP" = "1.26.4" ] && echo "PASS: prod numpy correct" || echo "FAIL"
[ "$TEST_NP" = "2.0.0" ] && echo "PASS: test numpy correct" || echo "FAIL"
```

### Break It on Purpose

**Install a package into system Python and see how it bleeds into everything:**

```bash
# DO NOT activate any venv
sudo pip3 install requests 2>/dev/null || pip3 install --user requests
# Now check: system requests is visible even inside a venv if --system-site-packages was used
python3.11 -m venv --system-site-packages venv-leaky
source venv-leaky/bin/activate
python3 -c "import requests; print('LEAKED:', requests.__version__)"
deactivate
```

**Diagnosis:** `--system-site-packages` makes system packages visible inside the venv. The deployment guide does NOT use this flag, so production venvs are fully isolated.

### Compare Approaches

| Approach | Isolation | Overhead | When to use |
|----------|-----------|----------|-------------|
| `venv` | Package-level | Minimal | Single-app servers (ZQL) |
| `conda` | Package + binary | Larger disk use | Data science, complex native deps |
| Docker container | Full OS-level | Higher | Multi-service, microservices |

### Stretch Challenges

1. Freeze the production venv to a `requirements.txt` and recreate it from scratch on a fresh venv.
2. Write a script that compares packages across two venvs and reports differences.
3. Create a Docker-based isolation setup and compare startup time vs. venv.

<details><summary>Hints</summary>

**Challenge 1:** `pip freeze > requirements.txt && python3.11 -m venv venv-rebuild && source venv-rebuild/bin/activate && pip install -r requirements.txt`
</details>

### Common Pitfalls

| Mistake | Symptom | Fix |
|---------|---------|-----|
| Forgetting to `source activate` | System Python runs instead | Check `which python3` |
| Using `pip` outside venv | Installs to system or user | Always activate first |
| Wrong Python version in venv | Import errors | Specify `python3.11 -m venv` explicitly |

### Reflection Questions

1. The deployment guide uses `pip install -e ".[prod]"`. What does the `-e` (editable) flag do and why use it in production?
2. If two Gunicorn workers share the same venv, can one corrupt the other's imports?
3. How would you handle a venv on a server with only 1 GB of disk free?

### Deliverable

Three isolated venvs (`venv-prod`, `venv-test`, `venv`) with verified independent package sets. Save `pip freeze` output from each to `~/zql-labs/lab04_venv_comparison.txt`.

---

## Lab 5: Secrets Management with .env Files

- **Concept:** Environment-variable-driven configuration for production secrets
- **Approach:** Build a config loader from scratch, test secret isolation, and verify file permissions
- **Difficulty:** Beginner | **Time:** 75 min | **Prerequisites:** Lab 4
- **Learning outcomes:**
  1. You will be able to create and secure a production .env file with correct permissions
  2. You will be able to build a Python config loader using python-dotenv
  3. You will be able to validate configuration at startup and fail fast on missing values
  4. You will be able to explain why secrets must never be committed to git
- **Why this matters:** The ZQL platform stores API keys (IonQ, IBM), secret keys, and CORS origins in `.env.production` (source doc §2.1). Leaking these to git or the web means unauthorized users can run quantum jobs on paid hardware at your expense.

### Theory Primer

The Twelve-Factor App methodology prescribes storing config in environment variables. A `.env` file is a convenient local representation that `python-dotenv` loads at startup:

```
.env.production          →  load_dotenv()  →  os.environ['SECRET_KEY']
(file, chmod 600)            (at startup)       (available to app)
```

The deployment guide (§2.1) uses `chmod 600` — only the file owner (deploy) can read it. No other user, no web server, no backup script can access secrets.

*(Ref: Source doc §2.1 Create Production .env, §2.2 Load Environment in App)*

### Setup

```bash
source ~/zql-labs/server_info.env
ssh -i ~/.ssh/zql_lab_key deploy@$ZQL_DROPLET_IP
cd ~/zql-labs
source venv/bin/activate
pip install python-dotenv --quiet
```

### Walkthrough

**Step 1: Create a production .env file with realistic config.**

```bash
cat > .env.production << 'EOF'
ZQL_BACKEND=local
ZQL_MAX_SHOTS=8192
ZQL_POOL_SIZE=4
ZQL_TIMEOUT=30.0
ZQL_ZNE=false
ZQL_LOG_LEVEL=INFO
SECRET_KEY=dev_only_change_me_in_production
ALLOWED_ORIGINS=https://quantumhub.zuntenium.com
EOF
```

**Step 2: Set restrictive permissions.**

```bash
chmod 600 .env.production
ls -la .env.production
```

Expected output:
```
-rw------- 1 deploy deploy ... .env.production
```

**Checkpoint:** Only owner `deploy` has read/write. No group or other access.

**Step 3: Build a config loader with validation.**

```bash
cat > config.py << 'PYEOF'
"""ZQL Configuration Loader — Lab 5"""
import os
import sys
from dotenv import load_dotenv
from dataclasses import dataclass
from typing import Optional

@dataclass
class ZQLConfig:
    backend: str
    max_shots: int
    pool_size: int
    timeout: float
    zne: bool
    log_level: str
    secret_key: str
    allowed_origins: list[str]
    ionq_api_key: Optional[str] = None
    ibm_api_key: Optional[str] = None

    @classmethod
    def from_env(cls, env_file: str = '.env.production') -> 'ZQLConfig':
        load_dotenv(env_file)

        required = ['ZQL_BACKEND', 'ZQL_MAX_SHOTS', 'SECRET_KEY']
        missing = [k for k in required if not os.getenv(k)]
        if missing:
            print(f"FATAL: Missing required config: {missing}", file=sys.stderr)
            sys.exit(1)

        return cls(
            backend=os.getenv('ZQL_BACKEND', 'local'),
            max_shots=int(os.getenv('ZQL_MAX_SHOTS', '4096')),
            pool_size=int(os.getenv('ZQL_POOL_SIZE', '2')),
            timeout=float(os.getenv('ZQL_TIMEOUT', '30.0')),
            zne=os.getenv('ZQL_ZNE', 'false').lower() == 'true',
            log_level=os.getenv('ZQL_LOG_LEVEL', 'INFO'),
            secret_key=os.getenv('SECRET_KEY'),
            allowed_origins=os.getenv('ALLOWED_ORIGINS', '').split(','),
            ionq_api_key=os.getenv('IONQ_API_KEY'),
            ibm_api_key=os.getenv('IBM_API_KEY'),
        )

    def validate(self) -> None:
        assert self.max_shots > 0, f"max_shots must be positive, got {self.max_shots}"
        assert self.pool_size > 0, f"pool_size must be positive, got {self.pool_size}"
        assert self.timeout > 0, f"timeout must be positive, got {self.timeout}"
        assert self.secret_key != 'dev_only_change_me_in_production', \
            "SECRET_KEY must be changed from default!"
        print("Config validation passed")

if __name__ == '__main__':
    cfg = ZQLConfig.from_env()
    print(f"Backend: {cfg.backend}")
    print(f"Max shots: {cfg.max_shots}")
    print(f"Pool size: {cfg.pool_size}")
    print(f"Origins: {cfg.allowed_origins}")
    print(f"IonQ key: {'set' if cfg.ionq_api_key else 'not set'}")
    cfg.validate()
PYEOF
```

**Step 4: Test the config loader.**

```bash
python3 config.py
```

Expected output:
```
Backend: local
Max shots: 8192
Pool size: 4
Origins: ['https://quantumhub.zuntenium.com']
IonQ key: not set
Config validation passed
```

Wait — validation should fail because SECRET_KEY is the default. Let's fix:

```bash
# Generate a real secret key
REAL_KEY=$(python3 -c "import secrets; print(secrets.token_hex(32))")
sed -i "s/dev_only_change_me_in_production/$REAL_KEY/" .env.production
python3 config.py
```

**Checkpoint:** "Config validation passed" now appears.

**Step 5: Verify .env is gitignored.**

```bash
echo ".env*" >> .gitignore
git init --quiet 2>/dev/null
git add -n .env.production 2>&1
```

Expected: The file should not be staged (gitignore blocks it).

### Verification

```bash
# Test 1: Permissions are correct
PERMS=$(stat -c '%a' .env.production 2>/dev/null || stat -f '%Lp' .env.production)
[ "$PERMS" = "600" ] && echo "PASS: permissions 600" || echo "FAIL: permissions $PERMS"

# Test 2: Config loads successfully
python3 -c "from config import ZQLConfig; c = ZQLConfig.from_env(); print('PASS' if c.max_shots == 8192 else 'FAIL')"

# Test 3: Missing key causes exit
echo "" > .env.test
python3 -c "from config import ZQLConfig; ZQLConfig.from_env('.env.test')" 2>&1 | grep -q "FATAL"
[ $? -eq 0 ] && echo "PASS: missing key detected" || echo "FAIL"
```

### Break It on Purpose

**Set permissions to world-readable and observe the risk:**

```bash
chmod 644 .env.production
ls -la .env.production   # Anyone can read!
# Any user or web process on the server can now read your API keys
chmod 600 .env.production   # Fix immediately
```

**Remove a required variable:**

```bash
cp .env.production .env.production.bak
sed -i '/SECRET_KEY/d' .env.production
python3 config.py   # Should print FATAL and exit
cp .env.production.bak .env.production   # Restore
```

### Compare Approaches

| Approach | Security | Complexity | When to use |
|----------|----------|------------|-------------|
| .env file (chmod 600) | Good | Simple | Single-server (ZQL deploy) |
| HashiCorp Vault | Excellent | Complex | Multi-service, enterprise |
| AWS/DO Secrets Manager | Good | Medium | Cloud-native deploys |
| systemd EnvironmentFile | Good | Simple | systemd-managed services |

### Stretch Challenges

1. Encrypt the .env file at rest using `gpg` and decrypt it only at deploy time.
2. Add environment-specific loading: load `.env.local` in development and `.env.production` in production based on a `ZQL_ENV` variable.
3. Build a config diff tool that compares `.env.production` against a `.env.example` template and flags missing keys.

<details><summary>Hints</summary>

**Challenge 3:** Read both files, parse key=value pairs, and set-difference the keys.
</details>

### Common Pitfalls

| Mistake | Symptom | Fix |
|---------|---------|-----|
| Committing .env to git | Secrets in repo history | Add `.env*` to `.gitignore` before first commit; use `git filter-branch` to purge |
| chmod 644 or 755 | Other users can read secrets | `chmod 600 .env.production` |
| Default SECRET_KEY in production | Weak/known secret | Generate with `openssl rand -hex 32` |

### Reflection Questions

1. Why does the deployment guide comment `.env.production` must NEVER be committed to git? What's the worst-case scenario?
2. What is the advantage of `ZQLConfig.from_env()` over reading `os.environ` directly throughout the codebase?
3. In a Zuntenium deployment for a Kenyan university, the server might be shared. How does `chmod 600` protect against other users on the same machine?

### Deliverable

`.env.production` with correct permissions (600), `config.py` with validated loading, and test output showing fail-fast on missing keys. Save to `~/zql-labs/lab05_config/`.

---
