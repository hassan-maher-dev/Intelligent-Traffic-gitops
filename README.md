# 🚦 Intelligent Traffic Management - GitOps Repository

This repository serves as the single source of truth for the **Intelligent Traffic Management System** infrastructure and application state. It follows the **GitOps methodology** using **ArgoCD** to automatically synchronize and deploy configurations to an AWS EKS (Elastic Kubernetes Service) cluster.

## 🏗️ Architecture Overview

The system is deployed using a Microservices architecture, separating the data generation (Collector) from the presentation layer (Dashboard). The entire cluster is managed declaratively:
- **Continuous Deployment (CD):** Managed by ArgoCD.
- **Traffic Management:** Handled by Nginx Ingress Controller (Single Entry Point).
- **Observability:** Monitored via the Kube-Prometheus-Stack (Prometheus & Grafana) with custom alerting rules.

---

## 📂 Repository Structure

The manifests are organized by their roles within the cluster:

### 1. Application Manifests (Traffic App)
- `namespace.yaml`: Creates the isolated `traffic-app-ns` namespace.
- `configmap.yaml`: Centralized configuration mapping (e.g., internal service URLs).
- `deployment-dashboard.yaml`: Deploys the Flask-based web dashboard.
- `deployment-collector.yaml`: Deploys the Python script simulating IoT traffic sensors.
- `service.yaml`: Configures a `ClusterIP` service for internal communication (FinOps approach to avoid extra LoadBalancer costs).

### 2. Traffic Routing (Ingress)
- `traffic-ingress.yaml`: Routes external traffic to the Dashboard. Includes a Catch-All rule and firewall-bypass custom domains.
- `argocd-ingress.yaml`: Exposes the ArgoCD UI securely.
- `grafana-ingress.yaml`: Exposes the Grafana Monitoring Dashboard.

### 3. Infrastructure & Observability (App of Apps Pattern)
- `nginx-ingress-app.yaml`: ArgoCD Application CRD to deploy the Nginx Ingress Controller via Helm.
- `prometheus-app.yaml`: ArgoCD Application CRD to deploy the Kube-Prometheus-Stack via Helm.
- `prometheus-alerts.yaml`: Custom Prometheus rules that trigger critical alerts if the Traffic Dashboard goes down.

---

## 💡 Key DevOps & FinOps Practices Implemented

1. **True GitOps & App of Apps:** Infrastructure components (Nginx, Prometheus) are not installed manually; they are defined as code and pulled automatically by ArgoCD.
2. **FinOps & Cost Optimization:** Used `ClusterIP` for application services instead of multiple AWS Classic/Network LoadBalancers, utilizing a single Nginx Ingress Controller as the gateway.
3. **Firewall Bypass Strategy:** Implemented intelligent Ingress routing rules to bypass local network restrictions (Newly Observed Domains) by utilizing custom trusted domain mapping via local `hosts` files.
4. **Proactive Monitoring:** Automated AlertManager rules directly injected into Prometheus via GitOps to monitor service health.

---

## 🚀 How It Works

1. The Jenkins CI pipeline builds the Docker images and updates the image tags inside `deployment-dashboard.yaml` and `deployment-collector.yaml`.
2. **ArgoCD** detects the commit in this repository.
3. ArgoCD automatically triggers a rolling update in the AWS EKS cluster to reflect the new state, ensuring zero downtime.