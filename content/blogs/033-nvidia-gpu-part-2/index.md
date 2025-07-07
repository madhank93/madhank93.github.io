+++
title = "Making my homelab server AI ready with Nvidia GPU — Part II"
description = "A End-to-End guide on GPU Passthrough"
date = 2025-07-06T00:00:00+00:00

[taxonomies]
tags = ["gpu", "homelab", "proxmox", "ai", "nvidia"]

[extra]
toc = true
+++

![Icons — flaticon](https://cdn-images-1.medium.com/max/3840/0*zim-0XEcp9-91Ilt.png)

Since my previous post on [GPU passthrough in Proxmox using Pulumi](https://medium.com/@madhankumaravelu93/gpu-pass-through-in-proxmox-using-pulumi-c1972e64a4bb), I’ve made a significant improvements in my homelab automation stack. This blog post covers the migration to the latest Ubuntu image, kernel upgrades, enhanced cloud-init automation, and the usage of workflow using justfile.

## Upgrading to the latest Kernel

The most notable change is the adoption of the Ubuntu 24.04 cloud image as the default VM template. This ensures improved hardware compatibility, and optimal support for AI workloads. The config.yml file now points directly to the official Ubuntu 24.04 cloud image.

```yaml
proxmox:
    username: root@pam
    endpoint: https://192.168.1.235:8006/
    nodename: proxmox
    image_url: https://cloud-images.ubuntu.com/releases/24.04/release/ubuntu-24.04-server-cloudimg-amd64.img
```
This change makes sures that all new VMs will boot with the latest LTS release and kernel, which is essential for modern NVIDIA drivers.

## Enhanced cloud-init

The cloud-init configuration has been modified to include,

* **NVIDIA driver auto install**: The `runcmd` section now leverages `ubuntu-drivers autoinstall` for automatic NVIDIA driver installation, ensuring GPU-enabled VMs are ready for AI workloads.

* **Kernel and header installation**: The script ensures the latest kernel and headers are present.

## Centralized configuration

To streamline the deployment process, all environment specific settings are now centralized in config.yml (for cluster and node definitions) and .env (for sensitive data like passwords) added to gitignore. Pulumi code dynamically loads these configurations at runtime.

* `config.yml`: Stores Proxmox credentials, endpoint, node name, and image URL.

* `.env`: Holds secrets such as the Proxmox password.

## VM creation with GPU passthrough

When provisioning a VM that requires GPU access, the Pulumi Go code dynamically adjusts the VM specification. By setting HasGPU: true and specifying PCIe IDs in your config.yml, the correct passthrough configuration is automatically applied during provisioning.

```go
if config.HasGPU {
    args.Bios = pulumi.String("ovmf")
    args.Machine = pulumi.String("q35")
    var hostpcis vm.VirtualMachineHostpciArray
    for i, pcieID := range config.PcieIDs {
        hostpcis = append(hostpcis, vm.VirtualMachineHostpciArgs{
            Device: pulumi.String("hostpci0"),
            Pcie:   pulumi.Bool(true),
            Id:     pulumi.String(pcieID),
            Xvga:   pulumi.Bool(i == 0),
        })
    }
    args.Hostpcis = hostpcis
}
```

## Automation with justfile

Using a [Justfile](https://github.com/casey/just) allows you to define repeatable commands,

```yml
deploy:
    pulumi up

preview:
    pulumi preview

destroy:
    pulumi destroy

refresh:
    pulumi refresh
```

* `just up` – Deploy or update the cluster

* `just destroy` – Tear down all the resources

## Output

The output of [nvidia-smi](https://docs.nvidia.com/deploy/nvidia-smi/index.html) confirms NVIDIA GPUs are correctly detected and operational within the VM. This utility provides real-time details on GPU status, driver versions, memory usage, and running processes.

![](https://cdn-images-1.medium.com/max/3254/1*-Of-6ku12kz9YYIm9S8XMg.png)

<div align="center">* * * *</div>

<center>

Originally published on [Medium](https://medium.com/@madhankumaravelu93/making-my-homelab-server-ai-ready-with-nvidia-gpu-part-ii-a6ef95b7282b)

🌟 🌟 🌟 **The source code for this blog post can be found here** 🌟🌟🌟

[GitHub - madhank93/homelab](https://github.com/madhank93/homelab)

</center>

**Reference**:

[1] [https://pve.proxmox.com/wiki/PCI_Passthrough](https://pve.proxmox.com/wiki/PCI_Passthrough#How_to_know_if_a_graphics_card_is_UEFI_.28OVMF.29_compatible)

[2] [https://docs.nvidia.com/deploy/nvidia-smi/index.html](https://docs.nvidia.com/deploy/nvidia-smi/index.html)

[3] [https://just.systems/man/en/](https://just.systems/man/en/)