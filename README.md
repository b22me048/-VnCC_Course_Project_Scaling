# 📝 Note-Taking App — Kubernetes Deployment

This guide explains how to deploy the Note-Taking microservices on **Minikube**, with **Prometheus** monitoring, a **MySQL** database, and **KEDA** autoscaling.

## 📦 Architecture

* **Microservices:** `add-note`, `update-note`, `delete-note`, `get-note`
* **Database:** MySQL (+ MySQL Exporter for metrics)
* **Observability:** Prometheus
* **Autoscaling:** KEDA (ScaledObjects per microservice)

## 📁 Repo Structure

```
.
├── deployments/
│   ├── add-note.yaml
│   ├── update-note.yaml
│   ├── delete-note.yaml
│   └── get-note.yaml
├── services/
│   ├── add-note-service.yaml
│   ├── update-note-service.yaml
│   ├── delete-note-service.yaml
│   └── get-note-service.yaml
├── prometheus/
│   ├── config-map.yaml
│   ├── rbac.yaml
│   ├── deployment.yaml
│   └── service.yaml
├── mysql-exporter.yaml
├── mysql-exporter-service.yaml
├── hpaAddNote.yaml
├── hpaUpdateNote.yaml
├── hpaDeleteNote.yaml
├── hpaGetNote.yaml
└── README.md
```

> ⚠️ **TODO:** This structure currently only includes the MySQL **exporter** (for Prometheus metrics), not the MySQL database itself. Add `mysql-deployment.yaml` and `mysql-service.yaml` (with a PVC for persistence) before relying on this for anything beyond local testing.

## ✅ Prerequisites

| Tool | Notes |
|---|---|
| [Minikube](https://minikube.sigs.k8s.io/docs/start/) | Local Kubernetes cluster |
| [kubectl](https://kubernetes.io/docs/tasks/tools/) | Matching version with your cluster |
| Docker / VM driver | Required by Minikube (Docker, HyperKit, VirtualBox, etc.) |
| [Helm](https://helm.sh/docs/intro/install/) (optional) | Recommended install method for KEDA |

---

## 🔧 Step 1: Install & Start Minikube

Install Minikube for your OS: https://minikube.sigs.k8s.io/docs/start/

Start the cluster:

```bash
minikube start
```

Enable the metrics server (needed for HPA-style metrics):

```bash
minikube addons enable metrics-server
```

---

## 🚀 Step 2: Deploy Microservices

```bash
kubectl apply -f deployments/add-note.yaml
kubectl apply -f deployments/update-note.yaml
kubectl apply -f deployments/delete-note.yaml
kubectl apply -f deployments/get-note.yaml
```

Or apply the whole folder at once:

```bash
kubectl apply -f deployments/
```

---

## 🌐 Step 3: Deploy Services for Microservices

```bash
kubectl apply -f services/add-note-service.yaml
kubectl apply -f services/update-note-service.yaml
kubectl apply -f services/delete-note-service.yaml
kubectl apply -f services/get-note-service.yaml
```

Or:

```bash
kubectl apply -f services/
```

---

## 📈 Step 4: Set Up Prometheus Monitoring

Apply in this order (RBAC and ConfigMap must exist before the deployment mounts them):

```bash
kubectl apply -f prometheus/config-map.yaml
kubectl apply -f prometheus/rbac.yaml
kubectl apply -f prometheus/deployment.yaml
kubectl apply -f prometheus/service.yaml
```

Open the Prometheus UI:

```bash
minikube service prometheus
```

---

## 🛢️ Step 5: Deploy MySQL Exporter

```bash
kubectl apply -f mysql-exporter.yaml
kubectl apply -f mysql-exporter-service.yaml
```

> This deploys the **exporter** that scrapes MySQL metrics for Prometheus — it assumes a MySQL instance is already reachable (configured via the exporter's connection env vars/secret). It does not deploy MySQL itself. See the TODO above.

---

## ⚖️ Step 6: Deploy Autoscaling with KEDA

Install KEDA if it isn't already in your cluster:

```bash
# Helm (recommended)
helm repo add kedacore https://kedacore.github.io/charts
helm repo update
helm install keda kedacore/keda --namespace keda --create-namespace
```

Or follow the official docs: https://keda.sh/docs/latest/install/

Verify KEDA is running:

```bash
kubectl get pods -n keda
```

Apply the ScaledObjects:

```bash
kubectl apply -f hpaAddNote.yaml
kubectl apply -f hpaUpdateNote.yaml
kubectl apply -f hpaDeleteNote.yaml
kubectl apply -f hpaGetNote.yaml
```

These scale each microservice based on the metrics/triggers configured in each ScaledObject (e.g., Prometheus query, queue length).

---

## 🔍 Verification

```bash
kubectl get pods -A
kubectl get deployments
kubectl get services
kubectl get scaledobjects        # KEDA-specific resource check
```

---

## 🛠️ Troubleshooting

| Symptom | Likely Cause | Fix |
|---|---|---|
| `ImagePullBackOff` | Using a locally-built image Minikube can't see | Run `eval $(minikube docker-env)` and rebuild, or push the image to a registry |
| Prometheus targets show `down` | RBAC applied after deployment, or ConfigMap path mismatch | Re-apply `rbac.yaml` and `config-map.yaml`, then restart the Prometheus pod |
| ScaledObject has no effect | KEDA not installed, or HPA already exists for same deployment | `kubectl get pods -n keda`; ensure no conflicting HPA targets the same deployment |
| `minikube service prometheus` doesn't open | Running in a remote/headless environment | Use `minikube service prometheus --url` and open the URL manually, or `kubectl port-forward` |
| MySQL exporter can't connect | No MySQL instance deployed/reachable yet | Deploy MySQL first (see TODO), confirm exporter's DSN/secret points to the right host |

---

## 🧹 Cleanup

```bash
kubectl delete -f deployments/
kubectl delete -f services/
kubectl delete -f prometheus/
kubectl delete -f mysql-exporter.yaml -f mysql-exporter-service.yaml
kubectl delete -f hpaAddNote.yaml -f hpaUpdateNote.yaml -f hpaDeleteNote.yaml -f hpaGetNote.yaml
minikube stop
```

---

## ✅ Summary

* **Microservices:** `add`, `delete`, `update`, `get`
* **Observability:** Prometheus
* **Database:** MySQL (exporter deployed; DB deployment pending — see TODO)
* **Autoscaling:** KEDA
