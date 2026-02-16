# Install Kubernetes Tools (kubeadm, kubelet, kubectl) – Ansible Tasks

These Ansible tasks set up the official Kubernetes package repository (v1.28) on a Debian/Ubuntu system and install the core Kubernetes CLI/tools while preventing accidental upgrades.

## What these tasks do

- **Remove any broken/old Kubernetes repo**  
  Deletes `/etc/apt/sources.list.d/kubernetes.list` if it already exists (to avoid conflicts with old/wrong repos).

- **Install required dependencies**  
  Ensures `apt-transport-https`, `ca-certificates`, `curl`, and `gpg` are installed (needed for secure HTTPS repos and key handling).

- **Create keyrings directory**  
  Makes `/etc/apt/keyrings` (modern location for signed-by keys).

- **Download & install Kubernetes GPG key**  
  Fetches the official signing key for v1.28 from `pkgs.k8s.io` and converts it to binary `.gpg` format (only runs if file doesn't exist).

- **Add the Kubernetes apt repository**  
  Creates `/etc/apt/sources.list.d/kubernetes.list` pointing to the stable v1.28 Debian repo, signed by the new key.

- **Update apt cache**  
  Refreshes package lists so the new repo is visible.

- **Install Kubernetes components**  
  Installs exactly:  
  - `kubeadm` – cluster bootstrap tool  
  - `kubelet` – node agent  
  - `kubectl` – command-line tool

- **Hold package versions**  
  Runs `apt-mark hold` on `kubelet`, `kubeadm`, and `kubectl` to prevent automatic upgrades (critical for cluster stability – Kubernetes upgrades should be deliberate).

## Summary

This is the **standard way** to install Kubernetes binaries from the official repository (recommended by Kubernetes docs since ~2022).  
It prepares the node for running `kubeadm init` (on control-plane) or `kubeadm join` (on workers).

After these tasks:
- You have a clean, pinned installation of Kubernetes v1.28 tools  
- Ready for the next steps: configuring container runtime, initializing/joining the cluster

Typically used in Ansible playbooks for kubeadm-based Kubernetes deployments.