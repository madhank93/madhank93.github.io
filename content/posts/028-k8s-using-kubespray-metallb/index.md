+++
title = "Configuring Kubernetes cluster in Proxmox using Kubespray and MetalLB — Part II"
description = "Deploying High-Availabile k8s cluster on Proxmox using Kubespray and MetalLB"
date = 2024-12-22T00:00:00+00:00

[taxonomies]
tags = ["homelab", "proxmox", "self-hosting", "k8s", "kubespray", "metallb",  "pulumi", "golang"]

[extra]
toc = true
+++


![Banner](https://cdn-images-1.medium.com/max/3840/1*k96Ygxvp-pTbZrfOJK1fVw.png)

In my previous article, we explored setting up a [Kubernetes (k8s) infrastructure in Proxmox using Pulumi and Cloud-init](https://medium.com/@madhankumaravelu93/setting-up-kubernetes-on-proxmox-with-cloud-init-and-pulumi-part-i-64e697bdfe98). In this article we look into how to configure that Kubernetes setup using [Kubespray](https://kubespray.io/) — an open-source solution for automating the deployment and management of production-ready Kubernetes clusters.

### What is Kubespray?

Kubespray is a project maintained by the Kubernetes SIG team that leverages Ansible playbooks to automate and manage Kubernetes clusters. It supports a variety of configurations,

* **Container Network Interface (CNI):** Calico, Flannel, Weave, and Cilium.

* **Container Runtime Interface (CRI):** containerd, CRI-O, gVisor, and Kata Containers.

* **Container Storage Interface (CSI):** AWS EBS, Azure Disk, and GCP Persistent Disk.

### Why you need MetalLB ?

[MetalLB](https://metallb.io/) allows you to create Kubernetes services of type LoadBalancer in a bare-metal cluster.

### Prerequisite Adjustments

Since Kubespray handles most Kubernetes configuration tasks, some of the steps from my original Cloud-init setup are now redundant. And also I considered using Nix to configure Kubernetes, I found it required a steeper learning curve than I’m ready for at this stage.

Let’s begin the setup;

### Clone the Kubespray Repository

Start by cloning the Kubespray repository and checking out the latest release tag, in this case, v2.26.0

```sh
git clone https://github.com/kubernetes-sigs/kubespray
git checkout tags/v2.26.0
```

### Install Ansible

To deploy Kubespray, install Ansible in a Python virtual environment

```sh
VENVDIR=kubespray-venv
python3 -m venv $VENVDIR
source $VENVDIR/bin/activate
pip install -U -r requirements.txt
```

### Generate Inventory

Kubespray includes a Python script to generate a host inventory file. Use it to create and tweak the necessary configurations.

```sh
pip install -U -r kubespray/contrib/inventory_builder/requirements.txt

CONFIG_FILE=inventory/proxmox/hosts.yaml python3 kubespray/contrib/inventory_builder/inventory.py \
    k8s-controller-ip,192.168.1.75 \
    k8s-worker1-ip,192.168.1.76 \
    k8s-worker2-ip,192.168.1.204
```

This script generates a host configuration file.

```yml
all:
    hosts:
    k8s-controller1:
        ansible_host: 192.168.1.76
        ip: 192.168.1.76
        access_ip: 192.168.1.76
    k8s-controller2:
        ansible_host: 192.168.1.75
        ip: 192.168.1.75
        access_ip: 192.168.1.75
    k8s-controller3:
        ansible_host: 192.168.1.204
        ip: 192.168.1.204
        access_ip: 192.168.1.204
    k8s-worker1:
        ansible_host: 192.168.1.72
        ip: 192.168.1.72
        access_ip: 192.168.1.72
    k8s-worker2:
        ansible_host: 192.168.1.73
        ip: 192.168.1.73
        access_ip: 192.168.1.73
    k8s-worker3:
        ansible_host: 192.168.1.74
        ip: 192.168.1.74
        access_ip: 192.168.1.74
    children:
    kube_control_plane:
        hosts:
        k8s-controller1:
        k8s-controller2:
        k8s-controller3:
    kube_node:
        hosts:
        k8s-controller1:
        k8s-controller2:
        k8s-controller3:
        k8s-worker1:
        k8s-worker2:
        k8s-worker3:
    etcd:
        hosts:
        k8s-controller1:
        k8s-controller2:
        k8s-controller3:
    k8s_cluster:
        children:
        kube_control_plane:
        kube_node:
    calico_rr:
        hosts: {}
```

### Configure Cluster Settings

Edit the cluster settings variables file to define key configurations, such as the Kubernetes version and add-ons like MetalLB.

```yml
kube_version: v1.30.0
helm_enabled: true
kube_proxy_strict_arp: true
metrics_server_enabled: true
cert_manager_enabled: true
ingress_nginx_enabled: true
metallb_enabled: true
metallb_speaker_enabled: true
metallb_config:
    address_pools:
    primary:
        ip_range:
        - 192.168.1.210-192.168.1.255
        auto_assign: true
        avoid_buggy_ips: true
    layer2:
    - primary
```

### Deploy the Cluster

Run the Ansible playbook to deploy the cluster. Depending on your environment, this may take 20–30 minutes.

```sh
ansible-playbook -i ../inventory/proxmox/hosts.yaml -e @../inventory/proxmox/cluster.yml --user=ubuntu --become --become-user=root cluster.yml
```

After a successful run, verify that all nodes are joined

```sh
kubectl get nodes -o wide
```
### Test the Cluster

Test your Kubernetes cluster by deploying an Nginx application and exposing it as a LoadBalancer service to validate MetalLB

```sh
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --port 80 --type LoadBalancer
kubectl get svc
```

Access the application using the external IP address returned by the `kubectl get svc` command.

<div align="center">* * * *</div>

<center>

Originally published on [Medium](https://medium.com/@madhankumaravelu93/configuring-kubernetes-cluster-in-proxmox-using-kubespray-and-metallb-part-ii-ac073aac3bf3)

🌟 🌟 🌟 **The source code for this blog post can be found here** 🌟🌟🌟

[GitHub - madhank93/pulumi-proxmox at main](https://github.com/madhank93/pulumi-proxmox/tree/main/proxmox-k8s-II/inventory/proxmox)

</center>

**Reference**:

[1] [https://kubespray.io/#/](https://kubespray.io/#/)

[2] [https://metallb.io/](https://metallb.io/)

[3] [https://youtu.be/2SmYjj-GFnE?si=xI7ToyNuVmikdm16](https://youtu.be/2SmYjj-GFnE?si=xI7ToyNuVmikdm16)