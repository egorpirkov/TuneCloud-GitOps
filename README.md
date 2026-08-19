# TuneCloud GitOps & Helm Chart ⛵️

This repository contains the **GitOps configuration** for deploying [TuneCloud](https://github.com/egorpirkov/TuneCloud) to a Kubernetes cluster.

The application is packaged as a **Helm Chart** and deployed through **ArgoCD** using a GitOps workflow.

---

## 🎯 Purpose

This repository is the **single source of truth for TuneCloud's Kubernetes deployment configuration**.

It contains:

* Helm templates for the application and supporting services
* Deployment and service configuration
* Ingress and persistent storage configuration
* Prometheus `ServiceMonitor` resources
* Application image versions and deployment parameters

Application source code is maintained separately in the [TuneCloud repository](https://github.com/egorpirkov/TuneCloud).

---

## 📁 Repository Structure

```text
.
└── tunecloud/
    ├── Chart.yaml
    ├── values.yaml
    ├── .helmignore
    └── templates/
        ├── client-*
        ├── server-*
        ├── postgres-*
        ├── covers-pvc.yaml
        ├── ingress.yaml
        └── grafana-ingress.yaml
```

---

## ☸️ Kubernetes Resources

The Helm Chart defines the resources required to run the TuneCloud stack.

### 🌐 Frontend

* `client-deployment.yaml` — deploys the Nginx server containing the compiled React SPA.
* `client-service.yaml` — provides internal networking for the frontend.
* `ingress.yaml` — routes external traffic to the frontend.

### ⚙️ Backend

* `server-deployment.yaml` — deploys the Node.js / Fastify API.
* `server-service.yaml` — provides internal networking for the backend.
* `server-monitor.yaml` — `ServiceMonitor` for scraping application metrics.

### 🗄️ PostgreSQL

* `postgres-deployment.yaml` — deploys PostgreSQL 16.
* `postgres-service.yaml` — provides internal database networking.
* `postgres-configmap.yaml` — contains the initial database schema.
* `postgres-pvc.yaml` — persistent storage for PostgreSQL data.

### 📊 Observability

* `postgres-exporter-deployment.yaml` — deploys PostgreSQL Exporter.
* `postgres-exporter-service.yaml` — provides internal networking for the exporter.
* `postgres-exporter-monitor.yaml` — `ServiceMonitor` for PostgreSQL metrics.
* `grafana-ingress.yaml` — exposes Grafana through an Ingress.

### 💾 Storage & Routing

* `covers-pvc.yaml` — persistent storage for downloaded album covers.
* `ingress.yaml` — routes `tunecloud.local` to the frontend and `tunecloud.local/api` to the backend.

---

## 🔄 GitOps Workflow

The repository is part of an automated CI/CD pipeline:

```text
TuneCloud
    │
    │ push to main
    ▼
GitHub Actions
    │
    ├── Build Docker images
    ├── Push images → GHCR
    └── Update image tags
            │
            ▼
     TuneCloud-GitOps
            │
            │ Git commit
            ▼
          ArgoCD
            │
            │ sync
            ▼
       Kubernetes (k3s)
```

### 1. Continuous Integration

A push to the `main` branch of the application repository triggers GitHub Actions.

The pipeline:

* builds the client and server Docker images;
* tags images with the short Git commit SHA;
* pushes the images to GitHub Container Registry.

### 2. GitOps Configuration Update

After successfully building the images, the CI pipeline uses `yq` to update the image tags in `tunecloud/values.yaml`.

The updated configuration is committed and pushed to this repository.

### 3. ArgoCD Deployment

ArgoCD monitors this repository and detects changes to the Helm configuration.

When a new commit is detected, ArgoCD synchronizes the desired state with the Kubernetes cluster and applies the updated workloads.

---

## 🛠️ Configuration

Deployment parameters are configured through `tunecloud/values.yaml`.

Example:

```yaml
global:
  domain: tunecloud.local
  grafanaDomain: grafana.tunecloud.local

server:
  image: ghcr.io/egorpirkov/tunecloud-server:abc1234
  replicas: 1

client:
  image: ghcr.io/egorpirkov/tunecloud-client:abc1234
  replicas: 1
```

Secrets are not stored in this repository. Required Kubernetes Secrets must be created in the cluster before deployment.

---

## 🚀 Manual Deployment

The chart can also be deployed directly with Helm without ArgoCD.

```bash
git clone https://github.com/egorpirkov/TuneCloud-GitOps.git
cd TuneCloud-GitOps

helm install tunecloud ./tunecloud \
  -n tunecloud \
  --create-namespace

kubectl get pods -n tunecloud
```

To update an existing installation:

```bash
helm upgrade tunecloud ./tunecloud -n tunecloud
```

---

## 📜 License

GPL v3.0
