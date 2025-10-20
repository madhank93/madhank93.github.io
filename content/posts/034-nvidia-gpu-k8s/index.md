+++
title = "Adding Nvidia GPU boost to Proxmox k8s using Pulumi and Kubespray"
description = "A step-by-step guide to leveraging Nvidia GPUs in Kubernetes"
date = 2025-07-17T00:00:00+00:00

[taxonomies]
tags = ["nvidia", "homelab", "proxmox", "gpu", "proxmox", "pulumi", "kubespray", "k8s", "ai"]

[extra]
toc = true
+++

![Homelab](https://cdn-images-1.medium.com/max/3840/1*P0y96o3ZRD_P6ywwEK3YCw.jpeg)

In my previous article I talked about [configuring GPU in VM level in Proxmox using GPU passthrough](https://medium.com/@madhankumaravelu93/making-my-homelab-server-ai-ready-with-nvidia-gpu-part-ii-a6ef95b7282b). However, the ultimate goal was to make this capability available to containerized workloads. In this post, I’ll walk you through on setting up the GPU enabled capability in the Kubernetes cluster.

## Pre-flight check:

To ensure secure and automated access to the newly provisioned VMs, password authentication for the default `ubuntu` user has been disabled and now it relies on SSH keys. Before deployment, update the `cloud-init.yml` file with own public SSH key. This `cloud-init` configuration automatically adds key to the VM upon creation, allowing to connect securely without a password.

## Updating cluster inventory

Inventory file has been simplified with the recent changes to accommodate the following features.

```ini
worker4 ansible_host=192.168.1.69 has_gpu=true node_labels="{'node-role.kubernetes.io/gpu': 'true'}"
```

* **has_gpu=true** custom variable that acts as a flag

* **node_labels** applies a label to the node

## Dynamic networking configuration

GPU node uses a different network interface name `enp6s18` than the other nodes `ens18`. Because [kube-vip](https://kube-vip.io/) maintains high availability, which needs to know the correct interface on each node.

Instead of a static configuration, a dynamic solution is implemented in values.yml file using a [Jinja2](https://jinja.palletsprojects.com/en/stable/) template.

```j2
kube_vip_interface: "{% if has_gpu | default(false) %}enp6s18{% else %}ens18{% endif %}"
```

It checks for the has_gpu variable defined in the inventory file and Ansible automatically assigns the correct network interface for Kube-VIP on each nodes.

## Deployment with Docker and Justfile

To improve consistency and simplify the deployment process, I’ve containerized the [Kubespray](https://kubespray.io/#/) execution using Docker. And to manage the Docker command, I’ve adopted a [Justfile](https://just.systems/man/en/).

```yml
run-kubespray:
    docker run --rm -it --mount type=bind,source="$(pwd)/k8s_cluster_config",dst=/config \
        --mount type=bind,source="${HOME}/.ssh/id_ed25519",dst=/root/.ssh/id_ed25519 \
        quay.io/kubespray/kubespray:v2.28.0 bash -c "ansible-playbook -i /config/inventory/hosts.ini -e @/config/values.yml cluster.yml"
```

Now, setting up the entire cluster deployment is reduced to a single command `just run-kubespray`.

## Cluster verification

After Kubespray successfully sets up cluster, configure [kubectl](https://kubernetes.io/docs/reference/kubectl/) on local machine. Copy the admin.conf file from a control plane node to local `~/.kube/config`.

```sh
ssh ubuntu@<control-plane-node-ip> 'sudo cat /etc/kubernetes/admin.conf' > ~/.kube/config
```

Next, edit `~/.kube/config` file. Find the `server` address and replace it with the load balancer IP defined in Kubespray `values.yml `file (e.g., `https://192.168.1.10:6443`). This ensures commands are sent to the highly-available endpoint instead of a single node.

![Nodes list](https://cdn-images-1.medium.com/max/5984/1*6T-aKxXw-wCwBEPjwssYeQ.png)

Make sure all the nodes are joined by running the following command `kubectl get nodes --show-labels`

## Installing NVIDIA GPU operator

The [NVIDIA GPU Operator](https://github.com/NVIDIA/gpu-operator) automates the management of all the necessary software components to provision GPUs in Kubernetes, including drivers, container runtimes, and monitoring tools . The easiest way to install it is with Helm.

```sh
helm repo add nvidia https://helm.ngc.nvidia.com/nvidia
helm repo update
```

Next, install the operator in a dedicated namespace.

```sh
helm install gpu-operator nvidia/gpu-operator \
    --namespace gpu-operator \
    --create-namespace \
    -f nvidia-setup/values.yml
```

This command deploys the operator, which will then automatically detect the GPU on the labeled worker node and installs all the necessary drivers and plugins.

## Verifying GPU Integration

Describe the node and look for `nvidia.com/gpu` in the labels and `nvidia.com/gpu` under the `Allocatable` resources section.

```sh
kubectl describe node k8s-worker4
```

Should see an output similar to this.

```yml
Allocatable:
    cpu:                3400m
    ephemeral-storage:  114953451738
    hugepages-1Gi:      0
    hugepages-2Mi:      0
    memory:             7237188Ki
    nvidia.com/gpu:     1
    pods:               110
```

To confirm that everything is working, run a simple test pod that requests GPU resources and executes the nvidia-smi command.

Create a file named gpu-test.yaml with the following content.

```yml
apiVersion: v1
kind: Pod
metadata:
    name: cuda-smi-test
spec:
    # Ensures the pod is only scheduled on a node with the GPU label
    nodeSelector:
    node-role.kubernetes.io/gpu: "true"
    restartPolicy: OnFailure
    containers:
    - name: cuda-test-container
        image: "nvidia/cuda:12.8.1-devel-ubuntu22.04"
        command: ["nvidia-smi"]
        resources:
        requests:
            cpu: "250m"
            memory: "512Mi"
        limits:
            nvidia.com/gpu: "1"
            cpu: "1"
            memory: "1Gi"
```

Deploying the pod

```sh
kubectl apply -f nvidia-setup/gpu-test.yaml
```

After a few moments, check the logs of the pod.

```sh
kubectl logs pods/cuda-smi-test
```

If the installation was successful, the output will be the familiar nvidia-smi report, showing the details of the GPU, but this time generated from within a Kubernetes pod . With these changes, my homelab is now fully equipped to run GPU-accelerated workloads directly within Kubernetes.

![nvidia-smi output](https://cdn-images-1.medium.com/max/3168/1*oq2U6YQfB-hLBI7xGHeJxw.png)

## Final thoughts

This post demonstrates how to build a Kubernetes cluster on Proxmox using a mutable infrastructure model. Future goal is to explore an immutable setup and I plan to experiment with [Talos](https://www.talos.dev/) to achieve this.


<div align="center">* * * *</div>

<center>

Originally published on [Medium](https://medium.com/@madhankumaravelu93/adding-nvidia-gpu-boost-to-proxmox-k8s-using-pulumi-and-kubespray-d5d9d3dace94)

🌟 🌟 🌟 **The source code for this blog post can be found here** 🌟🌟🌟

[GitHub - madhank93/homelab at v0.1.4](https://github.com/madhank93/homelab/tree/v0.1.4)

</center>

**Reference:**

[1] [https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/index.html#](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/index.html#)

[2] [https://pve.proxmox.com/pve-docs/pve-admin-guide.html#_cpu_type](https://pve.proxmox.com/pve-docs/pve-admin-guide.html#_cpu_type)

[3] [https://kubespray.io/#/docs/ansible/inventory](https://kubespray.io/#/docs/ansible/inventory)

[4] [https://www.hashicorp.com/en/resources/what-is-mutable-vs-immutable-infrastructure](https://www.hashicorp.com/en/resources/what-is-mutable-vs-immutable-infrastructure)
