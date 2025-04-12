Here’s a comprehensive `README.md` for your `nexus-k8s-demo` project:
Kubernetes + MetalLB + NGINX + Fake Domain
---

# Nexus Kubernetes Demo

This repository contains a complete setup for deploying a Nexus Repository Manager in a Kubernetes cluster, with persistent storage using NFS, secured access via Ingress with HTTPS, and integrated monitoring using Prometheus and Grafana.

## 📁 Project Structure

```
nexus-k8s-demo/
├── nexus-deployment.yaml        # Nexus app with resource limits, PVC, and Ingress
├── nexus-nfs-pv.yaml            # Persistent Volume configuration for NFS
├── nexus-nfs-pvc.yaml           # Persistent Volume Claim to mount NFS
├── nexus-ingress.yaml           # Ingress configuration with TLS (HTTPS)
├── nexus.local.crt              # Self-signed TLS certificate
├── nexus.local.key              # Private key for self-signed TLS
├── prometheus-setup.yaml        # Prometheus setup and configuration
├── grafana-setup.yaml           # Grafana setup and configuration
├── nexus-metrics-dashboard.json # Custom dashboard for Nexus metrics in Grafana
├── README.md                    # Instructions for setting up and deploying
```

---

## 🛠 Prerequisites

- Kubernetes cluster (Minikube or other)
- Docker & kubectl installed
- Helm (optional for Prometheus/Grafana)
- NGINX Ingress Controller
- NFS Server (for persistent storage)
- MetallLb installed


---

## 🚀 Deployment Steps

### 1. Start Minikube and Enable Ingress


minikube start --driver=docker
minikube addons enable ingress
```
Install MetalLB for Load Balancing
MetalLB provides external IPs for services running in local Kubernetes

> Ensure NGINX ingress controller is running:

kubectl get pods -n ingress-nginx
```

---

### 2. Set Up NFS Persistent Volume and Claim

Apply the NFS storage configuration:

```bash
kubectl apply -f nexus-nfs-pv.yaml
kubectl apply -f nexus-nfs-pvc.yaml
```

---

### 3. Deploy Nexus Repository

```bash
kubectl apply -f nexus-deployment.yaml
```

Wait until Nexus is running:

```bash
kubectl get pods
```

---

### 4. Configure Ingress with TLS

Place your self-signed cert and key in Kubernetes secrets:

```bash
kubectl create secret tls nexus-tls --cert=nexus.local.crt --key=nexus.local.key
```

Then apply the Ingress:

```bash
kubectl apply -f nexus-ingress.yaml
```

> Add the following to your `/etc/hosts` (Linux/Mac) or `C:\Windows\System32\drivers\etc\hosts` (Windows):

```
127.0.0.1 nexus.local
```

Now visit: [https://nexus.local](https://nexus.local)

---

🔹 Install Metrics Server
This is required for Kubernetes to measure CPU and memory usage.

kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml


kubectl get deployment metrics-server -n kube-system
kubectl top pod
If you see CPU/memory usage in kubectl top pod, you're good ✅

🔹 Create Horizontal Pod Autoscaler (HPA)
kubectl apply -f hpa-nexus.yaml



### 5. Set Up Monitoring with Prometheus and Grafana

```bash
kubectl apply -f prometheus-setup.yaml
kubectl apply -f grafana-setup.yaml
```

> Access Grafana via port-forward:

```bash
kubectl port-forward svc/grafana 3000:3000
```

Login:  
- **Username:** admin  
- **Password:** admin

Import the custom dashboard from `nexus-metrics-dashboard.json`.

---

## 📊 Nexus Metrics Dashboard

The included Grafana dashboard (`nexus-metrics-dashboard.json`) provides insights into Nexus resource usage, request rates, error counts, etc.

---

## 🔒 Security Notes

- The Ingress uses a self-signed TLS certificate for HTTPS.
- For production use, replace with a valid certificate from Let's Encrypt or another CA.
- Consider enabling authentication on the Nexus instance.

---

## ✅ Cleanup

To remove all components:

```bash
kubectl delete -f grafana-setup.yaml
kubectl delete -f prometheus-setup.yaml
kubectl delete -f nexus-ingress.yaml
kubectl delete -f nexus-deployment.yaml
kubectl delete -f nexus-nfs-pvc.yaml
kubectl delete -f nexus-nfs-pv.yaml
kubectl delete secret nexus-tls
```


## ✍️ Author

Njeck Cdric .