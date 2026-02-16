# Deploy Monitoring Stack (Prometheus + Grafana) – Ansible Tasks

These tasks ensure the Kubernetes API is ready, wait for nodes to become Ready, install Helm, add the Prometheus repo, and deploy the **kube-prometheus-stack** Helm chart (which includes Prometheus, Alertmanager, Grafana, and many exporters).

## What these tasks do

- **Wait for API server to respond**  
  Runs `kubectl get nodes` (with `KUBECONFIG=/etc/kubernetes/admin.conf`)  
  Retries 10 times (every 15 seconds) until it succeeds (`rc == 0`)  
  → Confirms the control plane is healthy after `kubeadm init`.

- **Wait for all nodes to be Ready**  
  Checks `kubectl get nodes | grep -v NotReady`  
  Retries 10 times (every 20 seconds) until success  
  → Ensures no nodes are in `NotReady` state before installing addons.

- **Install Helm (if missing)**  
  Downloads and runs the official `get-helm-3` script  
  Skips if `/usr/local/bin/helm` already exists (`creates` arg).

- **Add & update Prometheus Helm repository**  
  Adds `prometheus-community` repo  
  Runs `helm repo update` to refresh charts.

- **Install/upgrade kube-prometheus-stack**  
  Uses Helm to deploy (or upgrade) the chart:  
  - Release name: `monitoring`  
  - Namespace: `monitoring` (created if missing)  
  - Waits for resources to be ready (`--wait`)  
  - Timeout: 20 minutes  
  - Sets Grafana service to `NodePort` on port `32000`  
  Retries up to 2 times (every 30 seconds) until success.

## Summary

After these tasks:
- Kubernetes API and nodes are confirmed ready  
- Helm is installed and configured  
- **Prometheus + Grafana** (via kube-prometheus-stack) are running in the `monitoring` namespace  
- Grafana is accessible via NodePort `32000` (e.g., `http://<node-ip>:32000`)  

Default credentials:  
- Username: `admin`  
- Password: Retrieve with `kubectl get secret -n monitoring monitoring-grafana -o jsonpath="{.data.admin-password}" | base64 --decode`

This is a standard, production-ready monitoring setup for Kubernetes clusters — ready to scrape metrics from nodes, kubelet, cAdvisor, etcd, API server, and your applications.