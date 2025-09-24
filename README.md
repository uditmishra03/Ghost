&nbsp;
<p align="center">
  <a href="https://ghost.org/#gh-light-mode-only" target="_blank">
    <img src="https://user-images.githubusercontent.com/65487235/157884383-1b75feb1-45d8-4430-b636-3f7e06577347.png" alt="Ghost" width="200px">
  </a>
  <a href="https://ghost.org/#gh-dark-mode-only" target="_blank">
    <img src="https://user-images.githubusercontent.com/65487235/157849205-aa24152c-4610-4d7d-b752-3a8c4f9319e6.png" alt="Ghost" width="200px">
  </a>
</p>
&nbsp;

<p align="center">
    <a href="https://ghost.org/">Ghost.org</a> •
    <a href="https://forum.ghost.org">Forum</a> •
    <a href="https://ghost.org/docs/">Docs</a> •
    <a href="https://github.com/TryGhost/Ghost/blob/main/.github/CONTRIBUTING.md">Contributing</a> •
    <a href="https://twitter.com/ghost">Twitter</a>
    <br /><br />
    <a href="https://ghost.org/">
        <img src="https://img.shields.io/badge/downloads-100M+-brightgreen.svg" alt="Downloads" />
    </a>
    <a href="https://github.com/TryGhost/Ghost/releases/">
        <img src="https://img.shields.io/github/release/TryGhost/Ghost.svg" alt="Latest release" />
    </a>
    <a href="https://github.com/TryGhost/Ghost/actions">
        <img src="https://github.com/TryGhost/Ghost/workflows/CI/badge.svg?branch=main" alt="Build status" />
    </a>
    <a href="https://github.com/TryGhost/Ghost/contributors/">
        <img src="https://img.shields.io/github/contributors/TryGhost/Ghost.svg" alt="Contributors" />
    </a>
</p>

&nbsp;


<a href="https://ghost.org/"><img src="https://user-images.githubusercontent.com/353959/169805900-66be5b89-0859-4816-8da9-528ed7534704.png" alt="Fiercely independent, professional publishing. Ghost is the most popular open source, headless Node.js CMS which already works with all the tools you know and love." /></a>

&nbsp;
&nbsp;

# Problem Statement

Deploy and operate a **production-grade Ghost blogging platform** across staging and production environments with strong DevOps practices.

## Requirements

- **CI:** SonarQube for code quality checks.
- **CD:** Build and push Docker images to AWS ECR.
- **Deployments:** Docker Compose on remote EC2 servers.
- **Extras:** Smoke testing, monitoring, alerting, and rollback mechanism.

---

# Ghost CI/CD Setup

## Server Prep

- Provision staging, production, and DevOps servers.
- Install Docker + Docker Compose.
- Create working directory: `~/ghost`

## App Config

- Add `config.production.json`(code given below)
- Fix Ghost content folder permissions (code given below)

## Docker Compose

- Create:
  - `deploy/docker-compose.staging.yml`
  - `deploy/docker-compose.production.yml`
- Define Ghost + MySQL + volumes.
- SCP compose files from repo → servers.

## ECR & Image Build

- Setup ECR repo: `ghost-app`
- Build using Docker Buildx with caching.
- Push:
  - `:latest` (replace this with the version, don't use latest in production)
  - commit tag
  - `:previous` (for rollback)

## SonarQube

- Configure host URL and token.
- Scan only changed files.
- Exclude `node_modules`, `dist`, `coverage`.

## Secrets

- GitHub secrets for:
  - staging/prod hosts
  - users
  - SSH keys
- SonarQube credentials.
- Staging health URL.

## Deployment

- **Staging:** SCP compose → pull + `up -d`
- **Smoke test:** `curl` health URL until HTTP 200.
- **Production:** Gated with manual approval.

## Rollback

- Separate rollback workflow.
- Input: staging or production.
- Pull `:previous`, retag `:latest`, force recreate.

## Monitoring

- **Prometheus** + **Blackbox Exporter**.
- Probes staging & production URLs.
- Alerts routed to **Slack**.

---

# Breaking the Problem

1. **Prepare Metadata:** Get commit SHA, tags, versions.
2. **Quality Gate:** Analyze via SonarQube, block if critical.
3. **Build & Push:** Docker image to ECR.
4. **Staging Deploy:** Use Compose, SCP, deploy.
5. **Smoke Test:** Health check.
6. **Production Deploy:** Post-approval only.
7. **Rollback:** Use `:previous` tag.
8. **Monitoring:** Blackbox, Slack alerts.

---

# Installations

## Docker + Buildx

```bash
sudo apt-get update -y
sudo apt-get install -y ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo $VERSION_CODENAME) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update -y
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo usermod -aG docker $USER
newgrp docker
```

## AWS CLI

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version
```

## SonarQube (via Docker Compose)

```bash
mkdir -p ~/sonarqube/{data,extensions,logs}
cat > ~/sonarqube/docker-compose.yml <<'YML'
services:
  sonarqube:
    image: sonarqube:community
    container_name: sonarqube
    ports:
      - "9000:9000"
    environment:
      - SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true
    volumes:
      - ./data:/opt/sonarqube/data
      - ./extensions:/opt/sonarqube/extensions
      - ./logs:/opt/sonarqube/logs
    healthcheck:
      test: ["CMD", "wget", "-qO-", "http://localhost:9000/about"]
      interval: 10s
      timeout: 3s
      retries: 20
YML

sudo chown -R 1000:1000 ~/sonarqube/data ~/sonarqube/extensions ~/sonarqube/logs
docker compose -f ~/sonarqube/docker-compose.yml up -d
```

---

# Self Hosted GitHub Runner

```bash
sudo useradd -m -s /bin/bash github
sudo usermod -aG docker github         
sudo su - github

mkdir actions-runner && cd actions-runner
curl -o actions-runner-linux-x64-2.328.0.tar.gz -L https://github.com/actions/runner/releases/download/v2.328.0/actions-runner-linux-x64-2.328.0.tar.gz
echo "01066fad3a2893e63e6ca880ae3a1fad5bf9329d60e77ee15f2b97c148c3cd4e  actions-runner-linux-x64-2.328.0.tar.gz" | shasum -a 256 -c
tar xzf ./actions-runner-linux-x64-2.328.0.tar.gz
./config.sh --url https://github.com/<your-org>/<your-repo> --token <REPO_REGISTRATION_TOKEN>
```

---

# Ghost App Setup on Servers

```bash
mkdir -p ~/ghost && cd ~/ghost
mkdir content
sudo chown -R 1000:1000 ./content
chmod -R 755 ./content
```

## config.production.json (Staging)

```json
{
  "url": "http://<staging_host>",
  "server": { "host": "0.0.0.0", "port": 2368 },
  "database": {
    "client": "mysql",
    "connection": {
      "host": "ghost-mysql",
      "port": 3306,
      "user": "ghost",
      "password": "ghostpass", #this is a sample, pls make the password more complex or use secrets manager
      "database": "ghostdb"
    }
  },
  "paths": { "contentPath": "/var/lib/ghost/content" }
}
```

## config.production.json (Production)

```json
{
  "url": "http://<production_host>",
  "server": { "host": "0.0.0.0", "port": 2368 },
  "database": {
    "client": "mysql",
    "connection": {
      "host": "ghost-mysql",
      "port": 3306,
      "user": "ghost",
      "password": "ghostpass", #this is a sample, pls make the password more complex or use secrets manager
      "database": "ghostdb"
    }
  },
  "paths": { "contentPath": "/var/lib/ghost/content" }
}
```

---

# Monitoring Stack

## Prometheus

```bash
curl -LO https://github.com/prometheus/prometheus/releases/download/v2.54.1/prometheus-2.54.1.linux-amd64.tar.gz
tar -xvf prometheus-2.54.1.linux-amd64.tar.gz
sudo mv prometheus-2.54.1.linux-amd64 /usr/local/prometheus
```
## prometheus.yml

```yaml
global:
  scrape_interval: 15s

alerting:
  alertmanagers:
    - static_configs:
        - targets: ["localhost:9093"]

rule_files:
  - "alert-rules.yml"

scrape_configs:
  - job_name: "ghost-uptime"
    metrics_path: /
    params:
      module: [http_2xx]
    static_configs:
      - targets:
          - http://<staging host>    
          - http://<production host>   
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: localhost:9115
```


**Systemd Service**

```ini
[Unit]
Description=Prometheus
After=network.target

[Service]
User=root
ExecStart=/usr/local/prometheus/prometheus \
  --config.file=/usr/local/prometheus/prometheus.yml \
  --storage.tsdb.path=/usr/local/prometheus/data
Restart=always

[Install]
WantedBy=multi-user.target
```

## Blackbox Exporter

```bash
curl -LO https://github.com/prometheus/blackbox_exporter/releases/download/v0.25.0/blackbox_exporter-0.25.0.linux-amd64.tar.gz
tar -xvf blackbox_exporter-0.25.0.linux-amd64.tar.gz
sudo mv blackbox_exporter-0.25.0.linux-amd64 /usr/local/blackbox_exporter
```

## blackbox.yml

```yaml
modules:
  http_2xx:
    prober: http
    timeout: 5s
    http:
      valid_http_versions: [ "HTTP/1.1", "HTTP/2" ]
      valid_status_codes: ['2xx']
      method: GET
```

**Systemd Service**

```ini
[Unit]
Description=Blackbox Exporter
After=network.target

[Service]
User=root
ExecStart=/usr/local/blackbox_exporter/blackbox_exporter \
  --config.file=/usr/local/blackbox_exporter/blackbox.yml
Restart=always

[Install]
WantedBy=multi-user.target
```

## Alertmanager

```bash
curl -LO https://github.com/prometheus/alertmanager/releases/download/v0.27.0/alertmanager-0.27.0.linux-amd64.tar.gz
tar -xvf alertmanager-0.27.0.linux-amd64.tar.gz
sudo mv alertmanager-0.27.0.linux-amd64 /usr/local/alertmanager
```
## alertmanager.yml

```yaml
route:
  receiver: "slack"

receivers:
  - name: "slack"
    slack_configs:
      - send_resolved: true
        channel: "<slack channel name>"
        username: "PrometheusBot"
        api_url: "<slack channel webhook>"
        text: >
          *Alert:* {{ .CommonAnnotations.summary }}
          *Description:* {{ .CommonAnnotations.description }}
          *Severity:* {{ .CommonLabels.severity }}
```

**Systemd Service**

```ini
[Unit]
Description=Alertmanager
After=network.target

[Service]
User=root
ExecStart=/usr/local/alertmanager/alertmanager \
  --config.file=/usr/local/alertmanager/alertmanager.yml \
  --storage.path=/usr/local/alertmanager/data
Restart=always

[Install]
WantedBy=multi-user.target
```

---

# How to Improve Complexity Further

## Deployment & Infrastructure

- Add a Domain and HTTPS certs to the staging, production and sonarqube endpoints.
- Migrate from Docker Compose to Kubernetes (EKS or Kind).
- Use Infrastructure as Code (Terraform or Ansible).

## CI/CD Pipeline Improvements

- Blue-Green or Canary deployments.
- Extend rollback automation to handle failed smoke tests.

## Observability & Monitoring

- Grafana Dashboards.
- Centralized logging via ELK or Loki.
- Create an escalation matrix for alerting (Alert -> L1 -> L2 -> L3)

## Security

- Use AWS Secrets Manager or Vault.
- Sign Docker images 
- Integrate Snyk for vulnerability scanning.

## Performance & Resilience

- Load testing via k6 or Locust as a pre-prod pipeline gate.



Copyright (c) 2013-2025 Ghost Foundation - Released under the [MIT license](LICENSE).
Ghost and the Ghost Logo are trademarks of Ghost Foundation Ltd. Please see our [trademark policy](https://ghost.org/trademark/) for info on acceptable usage.
