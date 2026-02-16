# Cloud Application Platform on OpenStack Kubernetes with Heat and Ansible

## Overview
This repository provides a comprehensive framework for deploying a cloud application platform on OpenStack Kubernetes using Heat templates and Ansible roles. This README will guide you through the architecture, components, and workflow involved in using this platform.

## Architecture
This hybrid cloud setup combines a manually installed OpenStack IaaS foundation (orchestrated via Heat) with a container platform (Kubernetes) deployed via Ansible on top of OpenStack VMs. The focus is to run both traditional VM workloads and modern containerized microservices (including our web app) in the same environment.
Deployment Flow

### Manual installation
OpenStack control-plane and compute services (Keystone, Glance, Nova, Neutron, Horizon, Cinder, Swift, Nova Compute + KVM) are installed and configured **manually** on the physical servers or base VMs.

### Heat orchestration
**Heat** templates are used to provision and manage the OpenStack infrastructure resources:
- VMs for Kubernetes control-plane and worker nodes
- Neutron networks, subnets, routers, security groups
- Cinder volumes (if needed for persistent storage)
- Floating IPs, key pairs, etc.

Heat only creates the machines and networking — it does **not** install or configure the OpenStack software itself.

### Ansible deploys Kubernetes & Monitoring
After the VMs are up (created by Heat), **Ansible** targets those instances and installs a production-grade Kubernetes cluster:
- kubeadm / kops-style bootstrap or custom playbooks
- etcd (stacked or external)
- Control plane: kube-apiserver, controller-manager, scheduler, cloud-controller-manager (with OpenStack integration via openstack-cloud-provider)
- Worker nodes: kubelet, kube-proxy, container runtime = **Docker**
- Common addons: CNI (Calico / Flannel / Cilium), CoreDNS, metrics-server, etc.

**Prometheus** and **Grafana** are also deployed as Kubernetes pods using Ansible, with Prometheus scraping metrics from OpenStack services, nodes, Docker, and Kubernetes components for unified observability.

### Dockerizing & Deploying Our Web App Microservices
We use **Docker** to containerize the microservices of our web application and deploy them into Kubernetes pods, enabling scalable, cloud-native execution on the OpenStack-provisioned infrastructure.

### Summary of responsibilities

- **Manual** → OpenStack services installation & configuration  
- **Heat** → OpenStack infrastructure provisioning & orchestration (VMs, networks, storage, …)  
- **Ansible** → Kubernetes cluster + monitoring stack (Prometheus + Grafana) + microservices deployment  
- **Docker** → Container runtime for all pods  

This results in a classic OpenStack + Kubernetes setup where Heat manages the “machines” and Ansible manages the “Kubernetes cluster(s)” (including monitoring and our dockerized web app), while keeping the OpenStack control plane installation itself manual.

The architecture diagram outlines the interaction between these components:

![Architecture Diagram](Architecture.jpg)

## Heat Template
This Heat template provisions a basic Kubernetes-ready infrastructure on OpenStack:

- 1 master node (`k8s-master`)  
- 1 worker node (`k8s-worker`)  

## What it creates

- Persistent Cinder root volumes (30 GB default, bootable from Ubuntu 22.04)  
- Nova VMs using flavor `k8s`  
- Neutron ports on `selfservice` network with security group `k8s-secgrp`  
- Floating IPs from `provider` network (public access to both nodes)  

## Node preparation (via cloud-init)

- Creates user `k8suser` / password `123` with full sudo access  
- Disables swap permanently  
- Loads kernel modules: `overlay` + `br_netfilter`  
- Configures sysctl settings required for Kubernetes networking (bridge iptables, IP forwarding)  
- Installs **Prometheus Node Exporter v1.7.0** as a systemd service  
  → exposes host metrics (CPU, memory, disk, network…) on port 9100  

## Parameters

| Parameter          | Default              | Description                              |
|--------------------|----------------------|------------------------------------------|
| `keypair_name`     | `mykey`              | SSH keypair name to inject into VMs      |
| `root_volume_size` | `30`                 | Size of each root volume in GB           |
| `ubuntu_image`     | `Ubuntu 22.04 Jammy` | Glance image used to create boot volumes |

## Usage

1. Deploy the stack:
   ```bash
   openstack stack create -t k8s-nodes.yaml k8s-nodes

Retrieve floating IPs (from stack outputs or openstack server list):Bashopenstack stack show k8s-nodes -c outputs
SSH into nodes:Bashssh k8suser@<floating-ip-master>
# password: 123

## Ansible Roles
Ansible roles are structured sets of tasks, variables, files, and templates that are used to define the configuration of your applications and services. This repository includes the following roles:
- **web_server**: Installs and configures a web server (e.g., Nginx, Apache).
- **database**: Sets up a database service (e.g., MySQL, PostgreSQL).
- **load_balancer**: Configures a load balancer to distribute traffic to application instances.

Each role can be found in the `ansible/roles/` directory, along with a `README.md` file explaining its specific usage.

## Deployment Workflow
The deployment workflow consists of the following high-level steps:
1. **Prepare your OpenStack Environment**: Ensure you have access to an OpenStack instance and the relevant credentials.
2. **Modify Heat Templates**: Customize the Heat templates as per your application requirements.
3. **Deploy Resources using Heat**: Use the OpenStack CLI or dashboard to deploy the Heat templates.
4. **Configure Applications with Ansible**:
   - Clone this repository.
   - Update the inventory file to include the target instances.
   - Run the deployment playbook using Ansible:
     ```bash
     ansible-playbook -i inventory deploy.yml
     ```
5. **Verify Deployment**: Access the deployed applications via the load balancer or direct instance IP addresses.

## Conclusion
This repository serves as a baseline for deploying a cloud application platform using OpenStack, Kubernetes, Heat, and Ansible. For advanced configurations and customizations, feel free to modify the provided templates and roles.

## License
This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
