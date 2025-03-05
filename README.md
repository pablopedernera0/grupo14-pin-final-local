Local Kubernetes Monitoring Project
Prerequisites

Docker
Minikube
kubectl
Helm

Setup Steps

1. Install Minikube

# For Linux/macOS
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# For Windows (using PowerShell)
New-Item -Path 'C:\' -Name 'minikube' -ItemType Directory -Force
Invoke-WebRequest -OutFile 'C:\minikube\minikube.exe' -Uri 'https://storage.googleapis.com/minikube/releases/latest/minikube-windows-amd64.exe'

2. Start Minikube Cluster
bash
minikube start --driver=docker --memory=8192 --cpus=4

3. Install Helm
# Linux/macOS
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Windows
choco install kubernetes-helm

5. Install Nginx Ingress Controller
bash
helm install nginx-ingress nginx/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace

6. Install Prometheus
bash
# Create namespace
kubectl create namespace monitoring

# Install Prometheus
helm install prometheus prometheus-community/prometheus \
  --namespace monitoring \
  --set alertmanager.enabled=false \
  --set pushgateway.enabled=false

7. Install Grafana
bash
helm install grafana grafana/grafana \
  --namespace monitoring \
  --set persistence.enabled=true \
  --set adminPassword=admin \
  --set service.type=LoadBalancer

8. Create Ingress for Grafana
Create a file grafana-ingress.yaml:
yaml

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: grafana-ingress
  namespace: monitoring
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /grafana
        pathType: Prefix
        backend:
          service:
            name: grafana
            port:
              number: 80

Apply the Ingress:
bash
kubectl apply -f grafana-ingress.yaml

9. Access Services
bash
# Grafana admin password
kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode

# Port-forward services (if needed)
kubectl port-forward -n monitoring service/grafana 3000:80
kubectl port-forward -n monitoring service/prometheus-server 9090:80


10. Verify Installation
bash
kubectl get pods -n monitoring
kubectl get services -n monitoring


Cleanup
bash
# Remove everything
minikube delete

# Or remove specific components
helm uninstall grafana -n monitoring
helm uninstall prometheus -n monitoring
helm uninstall nginx-ingress -n ingress-nginx