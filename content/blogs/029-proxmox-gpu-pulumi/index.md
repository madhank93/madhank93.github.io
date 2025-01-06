+++
title = "GPU pass through in Proxmox using Pulumi"
description = "A practical guide to automating GPU passthrough with Proxmox and Pulumi"
date = 2024-12-29T00:00:00+00:00

[taxonomies]
tags = ["homelab", "proxmox", "self-hosting", "gpu", "pulumi", "golang"]

[extra]
toc = true
+++


![Banner](https://cdn-images-1.medium.com/max/3840/1*0RZrlCvUeUS4rDPuwb63Hg.png)

Setting up GPU passthrough in [Proxmox](https://www.proxmox.com/en/) using Infrastructure as Code (IaC) with [Pulumi](https://www.pulumi.com/) enables automated and reproducible VM deployments with direct GPU access.

### Prerequisites

Before implementing GPU passthrough, ensure your system meets these requirements,

* IOMMU/VT-d support, and

* UEFI BIOS settings enabled.

### Pulumi Implementation

Here’s the simplified version of how to implement GPU passthrough using Pulumi with Go language.

```go
vm.VirtualMachineArgs{
    NodeName: pulumi.String("pve"),
    Bios:    pulumi.String("ovmf"),
    Machine: pulumi.String("q35"),
    Hostpcis: vm.VirtualMachineHostpciArray{
        vm.VirtualMachineHostpciArgs{
            Device: pulumi.String("hostpci0"),
            Pcie:   pulumi.Bool(true),
            Id:     pulumi.String("0000:17:00.0"), // Your GPU PCI ID
        },
    },
}
```
The code above demonstrates the essential configuration for GPU passthrough,

* Use ***OVMF*** BIOS for UEFI support,

* ***Q35 machine type*** for modern PCI Express support, and

* Correct ***PCI ID*** for your GPU.

![VM — Hardware config](https://cdn-images-1.medium.com/max/6048/1*Kuz-mk7uTywaitJX2iN6lA.png)

To verify the GPU configurations run the following command sudo lshw -C display to show detailed information about the graphics card.

![Graphics card details](https://cdn-images-1.medium.com/max/2974/1*SMI56hi5lzqPUi9rK3HGng.png)

### Key Limitations of GPU Passthrough in Proxmox

* Consumer grade GPU limited to one per virtual machine when using PCIe passthrough.

* VMs must be shut down before switching GPU assignment

<div align="center">* * * *</div>

<center>

Originally published on [Medium](https://medium.com/@madhankumaravelu93/gpu-pass-through-in-proxmox-using-pulumi-c1972e64a4bb)

🌟 🌟 🌟 **The source code for this blog post can be found here** 🌟🌟🌟

[GitHub - madhank93/pulumi-proxmox at main](https://github.com/madhank93/pulumi-proxmox/tree/gpu_pass_through/proxmox-k8s)

</center>

**Reference**:

[1] [https://www.pulumi.com/registry/packages/proxmoxve/](https://www.pulumi.com/registry/packages/proxmoxve/)

[2] [https://registry.terraform.io/providers/bpg/proxmox/latest/docs/resources/virtual_environment_vm](https://registry.terraform.io/providers/bpg/proxmox/latest/docs/resources/virtual_environment_vm)

[3] [https://www.youtube.com/watch?v=_hOBAGKLQkI](https://www.youtube.com/watch?v=_hOBAGKLQkI)
