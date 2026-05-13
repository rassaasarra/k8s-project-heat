# k8s-project-heat — Kubernetes Infrastructure

This directory contains the Kubernetes manifests and Ansible playbooks used to deploy the MediSync platform on a Kubernetes cluster.

## Manifests (`k8s/`)

| File | Kind | Purpose |
|---|---|---|
| `backend-configmap.yaml` | ConfigMap | Environment configuration for the backend |
| `backend-deployment.yaml` | Deployment | Backend application (mono-backend) |
| `backend-service.yaml` | Service (LoadBalancer) | Exposes the backend on port 8080 |
| `frontend-service.yaml` | Service (LoadBalancer) | Exposes the frontend on port 80 |
| `backend-hpa.yaml` | HorizontalPodAutoscaler | Autoscales `mono-backend` — min 2 / max 5 replicas at 70 % CPU |

> **Prerequisite for HPA:** the [metrics-server](https://github.com/kubernetes-sigs/metrics-server) must be running in the cluster (`kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml`).

### Applying the core manifests

```bash
kubectl apply -f k8s/
```

---

## Optional validation manifests

The following files (`db-deployment.yaml`, `db-secret.yaml`, `db-service.yaml`, `db-pvc.yaml`) are included to demonstrate Kubernetes database and secret management patterns. They provision a standalone PostgreSQL instance with proper Secret, ClusterIP Service, and PersistentVolumeClaim resources.

**The current application backend is NOT wired to this database.** No database URL is injected into `backend-deployment.yaml`, and the existing frontend/backend behaviour is completely unaffected by these manifests.

| File | Kind | Description |
|---|---|---|
| `db-secret.yaml` | Secret | Placeholder credentials for the demo database (not production values) |
| `db-pvc.yaml` | PersistentVolumeClaim | 1 Gi volume for PostgreSQL data |
| `db-deployment.yaml` | Deployment | `demo-postgres` — PostgreSQL 15, reads credentials from `demo-db-secret` |
| `db-service.yaml` | Service (ClusterIP) | `demo-postgres-service` — cluster-internal access on port 5432 |

Apply order matters (Secret and PVC must exist before the Deployment):

```bash
kubectl apply -f k8s/db-secret.yaml
kubectl apply -f k8s/db-pvc.yaml
kubectl apply -f k8s/db-deployment.yaml
kubectl apply -f k8s/db-service.yaml
```

Or apply the whole directory at once (Kubernetes will handle dependency ordering via readiness):

```bash
kubectl apply -f k8s/
```

> **Note:** The credentials in `db-secret.yaml` are hardcoded demo values. Replace them with a proper secrets management solution (e.g., HashiCorp Vault, Sealed Secrets) before any production use.
