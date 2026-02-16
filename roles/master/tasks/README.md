# Kubernetes Cluster Initialization with kubeadm – Ansible Tasks Summary

These tasks safely bootstrap the Kubernetes control plane on the master node using **kubeadm**.

## Key Actions

- **Advertise API server address**  
  Uses the node's default IPv4 address (`{{ ansible_default_ipv4.address }}`) for the API server endpoint.

- **Set pod network CIDR**  
  Configures `--pod-network-cidr=192.168.0.0/16` (required for Calico networking).

- **Idempotent initialization**  
  Runs `kubeadm init` **only** if `/etc/kubernetes/admin.conf` does not already exist.

- **Ensure kubelet is active**  
  Starts and enables the `kubelet` service (the Kubernetes node agent).

- **Configure kubectl access for non-root user**  
  Copies `/etc/kubernetes/admin.conf` → `/home/k8suser/.kube/config`  
  Sets ownership to `k8suser:k8suser` so `k8suser` can run `kubectl` without sudo.

- **Wait for control plane readiness**  
  Retries `kubectl get nodes` (10 attempts, 15s delay) until the API server responds successfully.

- **Install Calico CNI**  
  Applies the official Calico v3.26.1 manifest:  
  ```bash
  kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/calico.yaml

  This sets up pod networking (matches the chosen CIDR).
