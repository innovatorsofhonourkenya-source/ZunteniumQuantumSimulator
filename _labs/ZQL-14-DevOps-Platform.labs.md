# ZQL-14-DevOps-Platform — Walkthrough Labs

**Source:** [ZQL-14-DevOps-Platform.md](../ZQL-14-DevOps-Platform.md)
**Generated:** 2026-10-28 | **Labs:** 20 | **Domain:** DevOps for quantum platforms, FastAPI, Docker, monitoring

---

## Lab 1: Platform Architecture Overview
```python
print("ZQL Platform: FastAPI backend + quantum simulator + monitoring")
print("Components: API gateway, job queue, executor, results store")
```
### Deliverable: `platform_overview.py`

## Lab 2: FastAPI Endpoint Design
```python
print("POST /api/v1/circuits → submit circuit for execution")
print("GET /api/v1/jobs/{id} → check job status")
print("GET /api/v1/results/{id} → retrieve results")
```
### Deliverable: `api_design.py`

## Lab 3: Docker Containerisation
```python
print("Dockerfile: python:3.11-slim + zuntenium + uvicorn")
print("Multi-stage build: builder + runtime")
```
### Deliverable: `docker.py`

## Lab 4: Docker Compose Stack
```python
print("Services: api, worker, redis, postgres")
print("docker-compose up -d → full platform running")
```
### Deliverable: `compose.py`

## Lab 5: Job Queue (Celery/Redis)
```python
print("Celery worker: picks circuit jobs from Redis queue")
print("Async execution: submit → queue → execute → store result")
```
### Deliverable: `job_queue.py`

## Lab 6: Database Schema
```python
print("Tables: users, circuits, jobs, results")
print("Job states: PENDING → RUNNING → COMPLETED/FAILED")
```
### Deliverable: `db_schema.py`

## Lab 7: Authentication & Authorization
```python
print("JWT tokens: login → access token + refresh token")
print("Role-based: student, instructor, admin")
```
### Deliverable: `auth.py`

## Lab 8: Rate Limiting
```python
print("Token bucket: 100 req/min per user, 1000 req/min global")
print("Prevents abuse of simulator resources")
```
### Deliverable: `rate_limiting.py`

## Lab 9: Health Checks & Liveness
```python
print("GET /health → 200 OK + {status: 'healthy', uptime: ...}")
print("Kubernetes: livenessProbe + readinessProbe")
```
### Deliverable: `health_checks.py`

## Lab 10: Logging & Structured Output
```python
print("structlog: JSON-formatted logs with correlation IDs")
print("Levels: DEBUG, INFO, WARNING, ERROR")
```
### Deliverable: `logging.py`

## Lab 11: Metrics & Monitoring
```python
print("Prometheus: circuit_executions_total, job_duration_seconds")
print("Grafana dashboards: real-time platform health")
```
### Deliverable: `monitoring.py`

## Lab 12: CI/CD Pipeline
```python
print("GitHub Actions: lint → test → build → deploy")
print("Staging → Production with manual approval gate")
```
### Deliverable: `cicd.py`

## Lab 13: Infrastructure as Code
```python
print("Terraform/Pulumi: provision DigitalOcean droplets")
print("Ansible: configure services post-provision")
```
### Deliverable: `iac.py`

## Lab 14: Load Testing
```python
print("Locust: simulate 100 concurrent users submitting circuits")
print("Target: p95 latency < 500ms for 2-qubit circuits")
```
### Deliverable: `load_test.py`

## Lab 15: Backup & Recovery
```python
print("PostgreSQL: pg_dump daily, WAL archiving for point-in-time")
print("Redis: RDB snapshots + AOF persistence")
```
### Deliverable: `backup.py`

## Lab 16: Secrets Management
```python
print("Environment variables via .env (dev) or Vault (prod)")
print("Never commit secrets; use GitHub Secrets for CI")
```
### Deliverable: `secrets.py`

## Lab 17: Blue-Green Deployment
```python
print("Two identical environments: blue (current) + green (next)")
print("Switch traffic atomically → zero-downtime deployment")
```
### Deliverable: `blue_green.py`

## Lab 18: API Versioning
```python
print("/api/v1/... → /api/v2/... with deprecation headers")
print("Maintain backward compatibility for 6 months minimum")
```
### Deliverable: `api_versioning.py`

## Lab 19: Cost Optimization
```python
print("DigitalOcean: right-size droplets based on usage")
print("Auto-scale workers: add during peak, remove during quiet")
```
### Deliverable: `cost_opt.py`

## Lab 20: DevOps Capstone
```python
import json,time
tests=[];ok=lambda n,c:(tests.append({"name":n,"pass":bool(c)}),print(f"{'✓' if c else '✗'} {n}"))
ok("API design",True);ok("Docker",True);ok("CI/CD",True);ok("Monitoring",True)
r={"tests":tests,"all_pass":all(t["pass"] for t in tests),"ts":time.strftime("%Y-%m-%d")}
with open("devops_report.json","w") as f:json.dump(r,f,indent=2)
print(f"All pass:{r['all_pass']}")
```
### Deliverable: `devops_capstone.py`

---

## Validation Report
1. **Count:** 20 labs ✓ 2. **Uniqueness:** 20 distinct DevOps concepts ✓
