# Clean & Prepare APT for Kubernetes Installation – Ansible Tasks

These Ansible tasks ensure a clean, conflict-free APT state before adding the official Kubernetes repository. They are especially useful when re-running playbooks, fixing broken installs, or preparing nodes that may have old/incompatible Kubernetes packages.

## What these tasks do

- **Wait for APT locks to be released**  
  Loops until all APT lock files are free (`/var/lib/dpkg/lock`, `/var/lib/apt/lists/lock`, `/var/cache/apt/archives/lock`).  
  Sleeps 2 seconds between checks.  
  Prevents "Could not get lock" errors during subsequent `apt` operations.  
  Marked as `changed_when: false` (never reports as changed).

- **Remove broken/old Kubernetes repo file**  
  Deletes `/etc/apt/sources.list.d/kubernetes.list` if it exists  
  → Avoids duplicate/conflicting repo entries from previous attempts.

- **Remove old Kubernetes GPG keyring**  
  Deletes `/etc/apt/keyrings/kubernetes-apt-keyring.gpg` if present  
  → Clears out any outdated or corrupted signing key.

- **Clean APT cache**  
  Runs `apt autoclean` and `apt autoremove`  
  Removes obsolete package files and unused dependencies to free space and reduce noise.

- **Update APT cache safely**  
  Runs `apt update` to refresh package lists  
  Done after cleaning, so the system sees the latest (and correct) repositories.

## Summary

This block is a **robust pre-install cleanup routine** that:
- Waits for any ongoing APT operations to finish  
- Removes old Kubernetes repo and key artifacts  
- Cleans up the package cache  
- Refreshes APT so the next steps (adding the new repo, installing kubeadm/kubelet/kubectl) run smoothly

Commonly used at the beginning of Kubernetes node setup playbooks to make them idempotent and resilient to partial/failed previous runs.