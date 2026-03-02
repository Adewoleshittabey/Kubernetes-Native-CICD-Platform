# 🏭 Industrial Grade Project 1 (IGP1)
### ABC Technologies — Full DevOps Pipeline


---

## 📋 Table of Contents

- [Overview](#-overview)
- [Architecture](#-architecture)
- [Technology Stack](#-technology-stack)
- [Pipeline Flow](#-pipeline-flow)
- [Phase Breakdown](#-phase-breakdown)
- [Service Access Points](#-service-access-points)
- [Repository Structure](#-repository-structure)
- [Getting Started](#-getting-started)
- [Key Design Patterns](#-key-design-patterns)
- [Live Metrics](#-live-metrics)
- [Documentation](#-documentation)

---

## 🔍 Overview

IGP1 is a **production-grade DevOps pipeline** built end-to-end. This is my key deliverable following the completion of the Post Graduate Program in DevOps by Edureka in collaboration with Purdue University. The project transforms a raw Java source repository into a fully automated, containerised, Kubernetes-orchestrated, and monitored platform — implementing all major DevOps disciplines across six progressive phases.

> **Application:** ABC Technologies Retail Portal  
> **Stack:** Java → Maven → Jenkins → Docker → Ansible → Kubernetes → Prometheus/Grafana  
> **Deployment Targets:** VM Tomcat · Docker Container · Kubernetes (2 replicas)  
> **Monitoring:** Real-time metrics via Prometheus & Grafana dashboards

---

## 🏗️ Architecture

![IGP1 Architecture](./Assets/IGP1_Architecture_Diagram.png)



---

## 🛠️ Technology Stack

| Category | Technology | Version | Role |
|----------|-----------|---------|------|
| **Source Control** | Git + GitHub | Latest | Code versioning and CI/CD trigger |
| **Build Tool** | Apache Maven | 3.8.7 | Compile, test, package WAR |
| **CI/CD** | Jenkins | Latest | Pipeline orchestration |
| **Containerisation** | Docker | 24.x | Application containerisation |
| **Automation** | Ansible | 2.16.3 | Infrastructure as Code |
| **Orchestration** | K3s (Kubernetes) | v1.34.3 | Container orchestration |
| **Package Manager** | Helm | v3.20.0 | Kubernetes application deployment |
| **Metrics** | Prometheus | v3.9.1 | Metrics collection and storage |
| **Visualisation** | Grafana | Latest | Dashboards and alerting |
| **Alerting** | Alertmanager | Latest | Alert routing and management |
| **App Server** | Apache Tomcat | 10.1 | Java web application server |
| **Runtime** | OpenJDK | 21 | Java execution environment |
| **OS** | Ubuntu | 24.04.3 LTS | Host operating system |

---

## 🔄 Pipeline Flow

```
Code Push to GitHub
        │
        ▼
┌───────────────┐
│   Checkout    │  Clone main branch via SSH deploy key
└──────┬────────┘
       │
       ▼
┌───────────────┐
│ Build & Test  │  mvn clean test → 4 unit tests pass
└──────┬────────┘
       │
       ▼
┌───────────────┐
│  Package WAR  │  mvn package → ABCtechnologies-1.0.war (6.9MB)
└──────┬────────┘
       │
       ▼
┌───────────────┐
│ Docker Build  │  docker build -t igp1-app:$BUILD_NUMBER
└──────┬────────┘
       │
       ▼
┌───────────────┐
│Ansible Deploy │  ansible-playbook deploy-k8s.yml
└──────┬────────┘
       │
       ├──► Import image to K3s containerd
       │
       ├──► Update deployment.yaml with new tag
       │
       ├──► kubectl apply (rolling update, zero downtime)
       │
       └──► Wait for 2/2 replicas ready
               │
               ▼
┌───────────────┐
│ Health Check  │  curl http://localhost:30080 → HTTP 200 ✓
└──────┬────────┘
       │
       ▼
┌───────────────┐
│   Prometheus  │  Scrapes pod metrics every 30s via ServiceMonitor
│   + Grafana   │  Real-time dashboards: CPU, memory, network, disk
└───────────────┘
```

---

## 📚 Phase Breakdown

### ✅ Phase 1 — Environment Setup, Build & Deploy to Tomcat
> *January 24 – 30, 2026*

Establishes the complete foundation. Ubuntu VM provisioned with Git, Java (OpenJDK 11), and Maven. Java application compiled, 4 unit tests executed and passed, and a 6.9MB WAR artifact produced. Apache Tomcat 10 installed and the application deployed and verified in browser.

**Key Outcomes:**
- `mvn clean test` → 4 tests, 0 failures
- `ABCtechnologies-1.0.war` (6.9MB) produced
- Application live at `http://10.214.99.251:8080/ABCtechnologies-1.0/`

---

### ✅ Phase 2 — Source Control with Git & GitHub
> *January 30 – February 1, 2026*

Source code placed under Git version control and pushed to GitHub. ED25519 SSH key pair generated for secure, passwordless authentication. `.gitignore` configured to exclude build artifacts. Repository becomes the single source of truth for all CI/CD automation.

**Key Outcomes:**
- Git repository initialised, `main` branch established
- ED25519 SSH key registered with GitHub
- Remote: `git@github.com:Gats-bey/Industrial-Grade-Project-1.git`

---

### ✅ Phase 3 — Jenkins CI/CD + Docker Integration
> *February 2 – 5, 2026*

**Part A:** Jenkins declarative pipeline automates the full build-to-Tomcat deployment cycle. A controlled, least-privilege deployment pattern is implemented — Jenkins stages the WAR to `/opt/jenkins-deploy`, and a root-owned script performs the actual Tomcat deployment via a restricted `sudoers` rule.

**Part B:** Docker containerisation introduced alongside Tomcat. Dockerfile created using `tomcat:10.1-jdk21-temurin` base image. Jenkins pipeline extended with Docker build and deploy stages. Application runs on port 8082 as a containerised service.

**Key Outcomes:**
- Full CI/CD pipeline: checkout → build → test → package → deploy → health check
- Dual deployment: Tomcat (`:8080`) + Docker (`:8082`)
- `igp1-app:10` Docker image (483MB) running

---

### ✅ Phase 4 — Ansible Automation
> *February 7 – 8, 2026*

Ansible introduced as Infrastructure as Code layer. Two playbooks created: `deploy-docker.yml` for Docker deployments and `deploy-k8s.yml` for Kubernetes deployments. Jenkins pipeline updated to trigger Ansible playbooks as the deployment mechanism, abstracting deployment logic from the pipeline itself.

**Key Outcomes:**
- Ansible playbooks in `/opt/ansible-igp1/`
- Jenkins pipeline: `ansible-playbook deploy-k8s.yml -e build_number=$BUILD_NUMBER`
- Idempotent, repeatable deployments

---

### ✅ Phase 5 — Kubernetes Orchestration (K3s)
> *February 10 – 12, 2026*

K3s single-node Kubernetes cluster installed. Kubernetes manifests created for Deployment (2 replicas), Service (NodePort:30080), and ServiceMonitor. Docker images imported to K3s containerd runtime via custom import script. Rolling updates enabled with zero downtime. Jenkins pipeline updated to support `kubernetes` as a deployment target.

**Key Outcomes:**
- K3s cluster: 1 node, `control-plane`, `Ready`
- 2 replicas running with liveness and readiness probes
- Rolling updates validated (6 ReplicaSets in history)
- `curl http://localhost:30080` → `Welcome to ABC technologies`

---

### ✅ Phase 6 — Prometheus & Grafana Monitoring
> *February 14 – 15, 2026*

Complete observability stack deployed via Helm (`kube-prometheus-stack`). Prometheus scrapes metrics from the Kubernetes cluster, Node Exporter, Kube State Metrics, and the IGP1 application pods via a custom ServiceMonitor. Grafana dashboards show real-time CPU, memory, network, and disk metrics. Both Grafana and Prometheus exposed via NodePort.

**Key Outcomes:**
- 6 monitoring components running in `monitoring` namespace
- Both IGP1 pods monitored: ~9m CPU, ~168Mi memory each
- Node metrics: 8% CPU, 79% memory utilisation
- Grafana accessible at `http://localhost:30683`

---

## 🌐 Service Access Points

| Service | URL | Notes |
|---------|-----|-------|
| **Application (Kubernetes)** | `http://localhost:30080` | Primary — 2 replicas, rolling updates |
| **Application (Docker)** | `http://localhost:8082` | Phase 3 — containerised deployment |
| **Application (Tomcat VM)** | `http://localhost:8080/ABCtechnologies-1.0/` | Phase 1/2 — traditional deployment |
| **Jenkins CI/CD** | `http://localhost:8081` | Pipeline management |
| **Grafana** | `http://localhost:30683` | Monitoring dashboards (admin) |
| **Prometheus** | `http://localhost:30808` | Metrics query interface |

---

## 📁 Repository Structure

```
IGP1/
├── src/
│   └── main/
│       ├── java/com/abc/
│       │   ├── RetailModule.java
│       │   ├── RetailDataImp.java
│       │   └── RetailAccessObject.java
│       └── webapp/
├── target/
│   └── ABCtechnologies-1.0.war        # Built by Maven (6.9MB)
├── k8s-manifests/
│   ├── deployment.yaml                 # K8s Deployment — 2 replicas
│   ├── service.yaml                    # NodePort Service — port 30080
│   └── servicemonitor.yaml             # Prometheus scrape config
├── Dockerfile                          # tomcat:10.1-jdk21-temurin base
├── Jenkinsfile                         # Declarative CI/CD pipeline
└── pom.xml                             # Maven build configuration

/opt/ansible-igp1/
├── deploy-docker.yml                   # Ansible Docker playbook
├── deploy-k8s.yml                      # Ansible Kubernetes playbook
└── inventory.ini                       # Ansible inventory

/usr/local/bin/
├── deploy_igp1.sh                      # Controlled Tomcat deploy script
└── k3s_import_igp1_image.sh           # Docker→K3s image import script
```

---

## 🚀 Getting Started

### Prerequisites

```bash
# Verify tools are installed
git --version
java -version
mvn -v
docker --version
kubectl get nodes
ansible --version
helm version
```

### Clone and Build

```bash
git clone git@github.com:Gats-bey/Industrial-Grade-Project-1.git
cd Industrial-Grade-Project-1
mvn clean test          # Run unit tests
mvn package             # Build WAR artifact
```

### Deploy via Jenkins

1. Open Jenkins at `http://localhost:8081`
2. Open the `igp1-ci` pipeline
3. Click **Build with Parameters**
4. Select `DEPLOYMENT_METHOD: kubernetes`
5. Click **Build**

Pipeline stages: Checkout → Build & Test → Package WAR → Docker Build → Ansible K8s Deploy → Health Check ✅

### Deploy Manually via Ansible

```bash
# Deploy to Kubernetes
sudo -u jenkins ansible-playbook /opt/ansible-igp1/deploy-k8s.yml \
  -i /opt/ansible-igp1/inventory.ini \
  -e build_number=<TAG> \
  -e project_directory=/var/lib/jenkins/workspace/igp1-ci

# Deploy to Docker
sudo -u jenkins ansible-playbook /opt/ansible-igp1/deploy-docker.yml \
  -i /opt/ansible-igp1/inventory.ini \
  -e build_number=<TAG>
```

### Check Running State

```bash
# Kubernetes
kubectl get pods -l app=igp1-app
kubectl get svc igp1-app-service
kubectl top pods

# Docker
docker ps
curl -I http://localhost:8082/

# Monitoring
kubectl get pods -n monitoring
```

---

## 🎯 Key Design Patterns

### Least-Privilege Deployment
Jenkins never has direct write access to Tomcat or Kubernetes system paths. All privileged operations are performed by controlled, root-owned scripts via a single restricted `sudoers` rule.

### Pipeline as Code
The complete CI/CD pipeline is defined in `Jenkinsfile`, committed to Git alongside source code. Every pipeline change is versioned and traceable.

### Infrastructure as Code
All deployments are managed via Ansible playbooks. No manual kubectl or Docker commands are required in production — Ansible ensures idempotent, repeatable deployments.

### Dual Deployment Support
The Jenkins pipeline supports four deployment methods via a single parameter: `kubernetes` · `ansible` · `docker-direct` · `vm-tomcat`. This enables gradual migration and easy rollback.

### Zero-Downtime Rolling Updates
Kubernetes rolling updates ensure the application remains available during every deployment. Liveness and readiness probes prevent traffic from reaching pods that aren't ready.

---

## 📊 Live Metrics

*Captured from `kubectl top` and Grafana dashboards:*

| Metric | Value |
|--------|-------|
| Pod CPU (igp1-app-pod-1) | 9m cores |
| Pod CPU (igp1-app-pod-2) | 9m cores |
| Pod Memory (pod-1) | 170 MiB |
| Pod Memory (pod-2) | 166 MiB |
| Node CPU utilisation | 8% (346m / 4 cores) |
| Node Memory utilisation | 79% (3116 MiB / 4 GiB) |
| Node Disk usage | 51.4% of 24 GiB |
| Prometheus scrape interval | 30 seconds |
| Active replicas | 2 / 2 |
| Deployment rollout history | 6 ReplicaSets |

---

## 📄 Documentation

Full phase-by-phase documentation is available for each stage of the project:

| Document | Phase | Coverage |
|----------|-------|---------|
| `IGP1_Phase1_Documentation.pdf` | Phase 1 | Environment setup, Maven build, Tomcat deployment |
| `IGP1_Phase2_Documentation.pdf` | Phase 2 | Git, GitHub, SSH authentication |
| `IGP1_Phase3_Documentation.pdf` | Phase 3 | Jenkins CI/CD pipeline + Docker integration |
| `IGP1_Phase4_Documentation.pdf` | Phase 4 | Ansible automation and IaC |
| `IGP1_Phase5_Documentation.pdf` | Phase 5 | Kubernetes orchestration with K3s |
| `IGP1_Phase6_Documentation.pdf` | Phase 6 | Prometheus & Grafana monitoring |

---

## 🏆 Project Outcome

IGP1 demonstrates a complete, production-grade DevOps pipeline covering every stage of the modern software delivery lifecycle:

```
Source Control → CI/CD → Containerisation → Automation → Orchestration → Observability
    GitHub     → Jenkins →    Docker      →  Ansible   →  Kubernetes  →  Prometheus/Grafana
```

The platform supports **automated builds on every commit**, **zero-downtime rolling deployments**, **self-healing containers**, **horizontal scaling**, and **real-time performance monitoring** — the full set of capabilities expected in a modern DevOps environment.

---

*Industrial Grade Project 1 · ABC Technologies · Adewole Shitta Bey · 2026*
