Kubernetes Deployment for Note-Taking App
This guide explains how to set up and deploy the Note-Taking microservices with Minikube, along with Prometheus monitoring, MySQL database, and KEDA autoscaling.

🔧 Step 1: Install & Start Minikube
✅ Install Minikube
Follow official instructions for your OS here: https://minikube.sigs.k8s.io/docs/start/

✅ Start Minikube
minikube start
(Optional) Enable the metrics server (needed for HPA):

minikube addons enable metrics-server
🚀 Step 2: Deploy Microservices
Navigate to the deployments/ folder and apply the deployment YAMLs:

kubectl apply -f deployments/add-note.yaml
kubectl apply -f deployments/update-note.yaml
kubectl apply -f deployments/delete-note.yaml
kubectl apply -f deployments/get-note.yaml
🌐 Step 3: Deploy Services for Microservices
Apply the service YAMLs inside the services/ folder:

kubectl apply -f services/add-note-service.yaml
kubectl apply -f services/update-note-service.yaml
kubectl apply -f services/delete-note-service.yaml
kubectl apply -f services/get-note-service.yaml
📈 Step 4: Set up Prometheus Monitoring
Navigate to the prometheus/ folder and apply the configurations in this order:

kubectl apply -f prometheus/config-map.yaml
kubectl apply -f prometheus/rbac.yaml
kubectl apply -f prometheus/deployment.yaml
kubectl apply -f prometheus/service.yaml
To access the Prometheus UI in the browser:

minikube service prometheus
This will open Prometheus in your default web browser.

🛢️ Step 5: Deploy MySQL Pod and Service
kubectl apply -f mysql-exporter.yaml
kubectl apply -f mysql-exporter-service.yaml
⚖️ Step 6: Deploy HPA with KEDA
Make sure KEDA is installed in your cluster. If not, follow: https://keda.sh/docs/latest/install/

Then apply the following KEDA ScaledObjects:

kubectl apply -f hpaAddNote.yaml
kubectl apply -f hpaDeleteNote.yaml
kubectl apply -f hpaUpdateNote.yaml
kubectl apply -f hpaGetNote.yaml
These will autoscale your microservices based on custom metrics or queue length, depending on your configuration.

✅ Summary
Microservices: add, delete, update, get
Observability: Prometheus
Database: MySQL
Autoscaling: KEDA
