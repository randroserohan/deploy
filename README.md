# Frozen Yogurt Website - DevOps CI/CD Project

## Project Overview

This project demonstrates a complete DevOps CI/CD workflow using:

- GitHub
- Jenkins
- Docker
- Nginx
- Trivy
- Prometheus
- Node Exporter
- Grafana

## CI/CD Pipeline

GitHub Push
    ↓
Jenkins
    ↓
Checkout
    ↓
Validation
    ↓
Docker Image Build
    ↓
Trivy Security Scan
    ↓
Backup
    ↓
Docker Deployment
    ↓
Health Check
    ↓
Rollback on Failure

## Monitoring

Node Exporter → Prometheus → Grafana

## Ports

| Service | Port |
|---|---|
| Jenkins | 8080 |
| Website | 8081 |
| Grafana | 3000 |
| Prometheus | 9090 |
| Node Exporter | 9100 |

## Features

- Automated GitHub-triggered deployment
- Website validation
- Docker image creation
- Trivy security scanning
- Container deployment
- Backup and rollback
- Website health check
- Server monitoring
- Grafana dashboard
- Prometheus metrics

## Dashboard

Grafana Node Exporter Full:
Dashboard ID: 1860

-------------------------------------------------------------**********************************-----------------------------------------------------------------------------

My DevOps Project Architecture:

Developer
    │
    │ git push
    ▼
 GitHub
    │
    │ Webhook
    ▼
 Jenkins
    │
    ├── Checkout
    ├── Validate
    ├── Docker Build
    ├── Trivy Scan
    ├── Backup
    ├── Deploy
    ├── Health Check
    └── Rollback
           │
           ▼
      Docker/Nginx
           │
           ▼
        Website


Monitoring:

Linux Server
     │
     ▼
Node Exporter :9100
     │
     ▼
Prometheus :9090
     │
     ▼
Grafana :3000
     │
     ▼
Dashboard + Alerts
