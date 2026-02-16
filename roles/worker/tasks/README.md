# Join Worker Node to Kubernetes Cluster – Ansible Tasks

These tasks get the join command from the master and run it on the worker to add it to the cluster.

## What they do

- **Get join command**  
  Runs on the master (`k8s-master`):  
  `kubeadm token create --print-join-command`  
  Stores the full `kubeadm join ...` command.

- **Join the cluster**  
  Executes the command on the worker.  
  Skips if `/etc/kubernetes/kubelet.conf` already exists (idempotent).

## Summary

Automatically joins the worker node to the Kubernetes cluster using a fresh token from the master.  
Safe to re-run; does nothing if the node is already joined.