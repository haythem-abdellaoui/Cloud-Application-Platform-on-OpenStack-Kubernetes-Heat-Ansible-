# Install & Configure containerd (Ansible Tasks)

These Ansible tasks install and configure **containerd** as the container runtime on a Debian/Ubuntu-based system, preparing it for use with Kubernetes (recommended since Kubernetes 1.24+ dropped direct Docker support).

## What these tasks do

- **Install containerd**  
  Installs the `containerd` package from apt repositories and updates the package cache first.

- **Create config directory**  
  Ensures `/etc/containerd/` exists.

- **Generate default configuration**  
  Runs `containerd config default` to output the full default config.toml and captures the output.

- **Write default config file**  
  Saves the generated default configuration to `/etc/containerd/config.toml`.

- **Enable systemd cgroup driver**  
  Changes `SystemdCgroup = false` → `SystemdCgroup = true` in the config file.  
  → This is **required** for Kubernetes when using the systemd cgroup driver (the default and recommended choice since Kubernetes 1.20+).

- **Restart and enable containerd**  
  Restarts the containerd service, enables it to start on boot, and ensures it's running.

## Why this matters for Kubernetes

- containerd is a lightweight, high-performance container runtime (extracted from Docker).  
- Enabling `SystemdCgroup = true` aligns containerd with Kubernetes' default cgroup driver (`systemd`), avoiding compatibility issues with kubelet.  
- These steps are a standard part of most Kubernetes bootstrap playbooks (e.g. kubeadm-based clusters).

## Summary

Installs containerd → generates default config → enables systemd cgroups → restarts service.

Run this before installing kubeadm, kubelet, and kubectl.