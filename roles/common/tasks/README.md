# Kubernetes Node OS Preparation (Ansible Tasks)

These tasks configure a Linux host to meet Kubernetes prerequisites.

### What they do

- **Disable swap**  
  Turns off swap immediately (`swapoff -a`) and comments out swap lines in `/etc/fstab` so it stays disabled after reboot.

- **Load kernel modules**  
  Adds `overlay` and `br_netfilter` to `/etc/modules-load.d/k8s.conf` (required for container storage and Kubernetes networking).

- **Set sysctl parameters**  
  Creates `/etc/sysctl.d/k8s.conf` with:  
  - `net.bridge.bridge-nf-call-iptables = 1`  
  - `net.bridge.bridge-nf-call-ip6tables = 1`  
  - `net.ipv4.ip_forward = 1`  
  Then applies them immediately with `sysctl --system`.

- **Disable IPv6** (optional)  
  Sets `net.ipv6.conf.all.disable_ipv6 = 1` to turn off IPv6 system-wide.

### Purpose

Prepares the node for Kubernetes installation by:
- Removing swap (Kubernetes requirement)  
- Enabling necessary kernel features for containers & CNI plugins  
- Configuring proper networking behavior  
- (Optionally) avoiding IPv6-related issues

