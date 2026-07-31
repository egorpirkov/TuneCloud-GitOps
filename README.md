# TuneCloud GitOps & Helm Chart ⛵️

This repository contains the **GitOps** configuration for deploying the [TuneCloud](https://github.com/egorpirkov/TuneCloud) application to a Kubernetes cluster. 

The configuration is packaged as a **Helm Chart** and is designed to be automatically synchronized and deployed by **ArgoCD** following the GitOps methodology.

---

## 📁 Repository Structure

```
.
└── tunecloud/                      # Helm Chart directory
    ├── Chart.yaml                  # Chart metadata (version, app version)
    ├── values.yaml                 # Default configuration values (image tags, domains)
    ├── .helmignore
    └── templates/                  # Kubernetes manifests
        ├── client-*                # Frontend resources
        ├── server-*                # Backend API resources
        ├── postgres-*              # Database & Exporter resources
        ├── covers-pvc.yaml         # Storage for music covers
        ├── ingress.yaml            # Main application routing
        └── grafana-ingress.yaml    # Monitoring routing
```

---

## ☸️ Kubernetes Resources (Manifests)

The `templates/` directory defines all necessary Kubernetes entities to run the full TuneCloud stack in production.

### 🌐 Frontend (React / Nginx)
- **`client-deployment.yaml`**: Deploys the Nginx web server containing the compiled React SPA.
- **`client-service.yaml`**: Exposes the frontend pods internally (NodePort/ClusterIP).

### ⚙️ Backend API (Node.js / Fastify)
- **`server-deployment.yaml`**: Deploys the Node.js backend. Mounts necessary secrets and configuration for Spotify API, JWT, and Database connections.
- **`server-service.yaml`**: Internal networking for the backend.
- **`server-monitor.yaml`**: A `ServiceMonitor` Custom Resource (CRD) that tells the Prometheus Operator to scrape `/metrics` from the Fastify server.

### 🗄️ Database (PostgreSQL)
- **`postgres-deployment.yaml`**: Deploys the PostgreSQL 16 database.
- **`postgres-service.yaml`**: Exposes the database to the backend and exporter.
- **`postgres-configmap.yaml`**: Contains `init.sql` for automatic schema creation and indexing on first boot.
- **`postgres-pvc.yaml`**: PersistentVolumeClaim ensuring database data survives pod restarts.

### 📊 Observability (PostgreSQL Exporter)
- **`postgres-exporter-deployment.yaml`**: A sidecar-like exporter that translates Postgres metrics to Prometheus format.
- **`postgres-exporter-service.yaml`**: Internal networking for the exporter.
- **`postgres-exporter-monitor.yaml`**: `ServiceMonitor` to scrape database metrics.
- **`grafana-ingress.yaml`**: Ingress rules for accessing the Grafana dashboards.

### 💾 Shared Storage & Routing
- **`covers-pvc.yaml`**: PersistentVolumeClaim to store downloaded album covers persistently.
- **`ingress.yaml`**: Core routing configuration mapping `tunecloud.local` to the client, and `tunecloud.local/api` to the backend server.

---

## 🔄 The GitOps Workflow (CI/CD)

This repository is the single source of truth for the cluster state. It operates within a fully automated GitOps pipeline:

1. **Continuous Integration**: When application code is pushed to the main TuneCloud repo, GitHub Actions builds new Docker images and pushes them to `ghcr.io`.
2. **Configuration Update**: The CI pipeline automatically runs `yq` to patch the image tags in `tunecloud/values.yaml` and pushes a new commit to *this* repository.
3. **Continuous Delivery**: **ArgoCD**, running inside the Kubernetes cluster, detects the new commit in this repository. It automatically synchronizes the new Helm Chart state, rolling out the updated Pods seamlessly.

---

## 🛠️ Configuration (`values.yaml`)

The `values.yaml` file exposes the configurable parameters of the chart. It typically looks like this:

```yaml
global:
  domain: tunecloud.local
  grafanaDomain: grafana.tunecloud.local

server:
  image: ghcr.io/egorpirkov/tunecloud-server:sha-latest
  replicas: 1

client:
  image: ghcr.io/egorpirkov/tunecloud-client:sha-latest
  replicas: 1
```

*(Note: Secrets are managed outside of this repository for security reasons and must be injected into the cluster separately as Kubernetes Secrets).*

---

## 🚀 Manual Deployment

If you need to deploy this chart manually (without ArgoCD), you can use Helm directly:

```bash
# 1. Clone the GitOps repository
git clone https://github.com/egorpirkov/TuneCloud-GitOps.git
cd TuneCloud-GitOps

# 2. Install the Helm Chart
helm install tunecloud ./tunecloud -n tunecloud --create-namespace

# 3. Check deployment status
kubectl get pods -n tunecloud
```

To update a manual deployment:
```bash
helm upgrade tunecloud ./tunecloud -n tunecloud
```
