+++
title = "Deploying OpenAI’s GPT-OSS model on Kubernetes with Ollama"
description = "A step-by-step guide to self-hosting LLMs in a homelab"
date = 2025-08-10T00:00:00+00:00

[taxonomies]
tags = ["nvidia", "homelab", "ollama", "gpu", "proxmox", "k8s", "openai", "ai"]

[extra]
toc = true
+++

![Banner](https://cdn-images-1.medium.com/max/3840/1*egjv9_E9eIkSWJP-s2OARQ.jpeg)

Following up on the previous article about [setting up a home lab with AI capabilities](https://medium.com/@madhankumaravelu93/adding-nvidia-gpu-boost-to-proxmox-k8s-using-pulumi-and-kubespray-d5d9d3dace94), in this article we will cover the practical steps of running a large language model [gpt-oss](https://github.com/openai/gpt-oss) on a [Kubernetes](https://kubernetes.io/) cluster using [Ollama](https://ollama.com/) and all running within a [Proxmox](https://www.proxmox.com/en/) environment.

## Setting up Ollama

The deployment of Ollama within a k8s environment is simplified by using the [otwld/ollama](https://github.com/otwld/ollama-helm) helm chart. This chart manages various parameters, including GPU enablement for Nvidia, ingress configuration, and pulling various models from the library.

### Configure the values.yml

First, create a values.yml file to define the specific configuration. And it includes the instruction on how to set up Ollama, enabling GPU support, and specifies which model to download.

```yml
ollama:
    gpu:
    enabled: true
    type: "nvidia"
    number: 1
    models:
    pull:
        - gpt-oss:20b
ingress:
    annotations:
    nginx.ingress.kubernetes.io/proxy-read-timeout: "600"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "600"
    nginx.ingress.kubernetes.io/proxy-connect-timeout: "600"
    enabled: true
    className: "nginx"
    hosts:
    - host: ollama.local.com
        paths:
        - path: /
            pathType: Prefix
```

### Install the Helm Chart

With the values.yml file created, run the following helm command to install Ollama.

```sh
helm install ollama otwld/ollama \
    --namespace ollama \
    --create-namespace \
    -f values.yml
```

### Verifying the Installation

After the installation completed, verify that all components are running by listing the resources in the ollama namespace.

`kubectl get all -n ollama`

Should see an output similar to this,

```
NAME                        READY   STATUS    RESTARTS   AGE
pod/ollama-8699ddb5-c6n9q   1/1     Running   0          31m

NAME             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)     AGE
service/ollama   ClusterIP   10.233.62.132   <none>        11434/TCP   31m

NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/ollama   1/1     1            1           31m

NAME                              DESIRED   CURRENT   READY   AGE
replicaset.apps/ollama-8699ddb5   1         1         1       31m
```

To access the Ollama API using the hostname defined in the ingress, add the following entry to local `/etc/hosts` file, replacing with the IP address of Kubernetes ingress controller.

    INGRESS_IP_ADDRESS   ollama.local.com

## Testing the Deployed Model

Test the model by sending a request to the Ollama API endpoint using curl.

```sh
curl http://ollama.local.com/api/generate -d '{
    "model": "gpt-oss:20b",
    "prompt": "Why is the sky blue?",
    "stream": false
}'
```

If everything is configured correctly, a JSON response from the model will be sent.

## Monitoring resources

### Checking the Pod Logs

The pod logs provides the detailed information about the server’s status, including GPU detection and model loading.

    time=2025-08-10T12:58:29.372Z level=INFO source=types.go:130 msg="inference compute" id=GPU-075d926f-8a44-1552-2cbc-276dc5bfd68d library=cuda variant=v12 compute=12.0 driver=12.8 name="NVIDIA GeForce RTX 5070 Ti" total="15.5 GiB" available="15.3 GiB"
    time=2025-08-10T13:02:21.739Z level=INFO source=ggml.go:378 msg="offloaded 22/25 layers to GPU"

### Monitoring GPU Usage

While a query is being processed, SSH into the GPU enabled worker node and monitor the resource utilization in real time using the following command.

`watch -n 1 nvidia-smi`

```
Every 1.0s: nvidia-smi                                   worker4: Sun Aug 10 13:46:00 2025

Sun Aug 10 13:46:00 2025
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 570.133.07             Driver Version: 570.133.07     CUDA Version: 12.8     |
|-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA GeForce RTX 5070 Ti     Off |   00000000:01:00.0 Off |                  N/A |
|  0%   41C    P1             45W /  300W |   12678MiB /  16303MiB |      0%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+

+-----------------------------------------------------------------------------------------+
| Processes:                                                                              |
|  GPU   GI   CI              PID   Type   Process name                        GPU Memory |
|        ID   ID                                                               Usage      |
|=========================================================================================|
|    0   N/A  N/A         2506171      C   /usr/bin/ollama                       12668MiB |
+-----------------------------------------------------------------------------------------+

```

The above output shows the Ollama process consuming GPU resource.

<div align="center">* * * *</div>

<center>

Originally published on [Medium](https://medium.com/@madhankumaravelu93/deploying-openais-gpt-oss-model-on-kubernetes-with-ollama-dec1efb158fb)

</center>

**Reference:**

[1] [https://github.com/otwld/ollama-helm](https://github.com/otwld/ollama-helm)

[2] [https://github.com/openai/gpt-oss](https://github.com/openai/gpt-oss)

[3] [https://enterprise-support.nvidia.com/s/article/Useful-nvidia-smi-Queries-2](https://enterprise-support.nvidia.com/s/article/Useful-nvidia-smi-Queries-2)
