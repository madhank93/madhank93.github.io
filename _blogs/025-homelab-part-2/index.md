+++
title = "Building a Home Lab server — Part II"
description = "Automating Proxmox VM Deployment using Pulumi and Go"
date = 2024-10-20T00:00:00+00:00

[taxonomies]
tags = ["homelab", "proxmox", "self-hosting", "servers", "pulumi", "golang"]

[extra]
toc = true
+++

![Go, Proxmox & Pulumi](https://cdn-images-1.medium.com/max/3840/1*wUE9SYSpu4rdpe7ploKHzg.png)

In this blog post, we will take a look at the [Golang](https://go.dev/) based script using [Pulumi](https://www.pulumi.com/), an Infrastructure-as-Code (IaC) tool, to automate the process of creating a virtual machine (VM) in a [Proxmox Virtual Environment ](https://www.proxmox.com/en/proxmox-virtual-environment/overview)(Proxmox VE).

The following example uses Pulumi package available at [https://github.com/muhlba91/pulumi-proxmoxve](https://github.com/muhlba91/pulumi-proxmoxve), which is a Pulumi provider for Proxmox VE. The provider itself is built on top of the Terraform provider for Proxmox, found at [https://github.com/bpg/terraform-provider-proxmox](https://github.com/bpg/terraform-provider-proxmox).

## Code Walkthrough

### 1. Loading Environment Variables

The script starts by loading environment variables from a `.env` file using the [godotenv](https://github.com/joho/godotenv) package.

`.env` file

```ini
PROXMOX_USERNAME=XXXXXXXXXXXX
PROXMOX_PASSWORD=XXXXXXXXXXXX
```

`main.go` file

```go
err := godotenv.Load()
```
It then reads the Proxmox username and password from the environment. These values are used to authenticate with the Proxmox server.

```go
proxmoxUsername := os.Getenv("PROXMOX_USERNAME")
proxmoxPassword := os.Getenv("PROXMOX_PASSWORD")

if proxmoxUsername == "" || proxmoxPassword == "" {
    return fmt.Errorf("PROXMOX_USERNAME and PROXMOX_PASSWORD must be set in the .env file")
}
```

### 2. Configuring the Proxmox Provider

The next step is setting up the Proxmox provider, **Endpoint** specifies the URL for the Proxmox server, **Username** and **Password** are used for authentication, **Insecure**whether to skip the TLS verification step.

```go
provider, err := proxmoxve.NewProvider(ctx, "proxmoxve", &proxmoxve.ProviderArgs{
    Endpoint: pulumi.String("https://192.168.1.198:8006/"),
    Username: pulumi.String(proxmoxUsername),
    Password: pulumi.String(proxmoxPassword),
    Insecure: pulumi.Bool(true),
})
```

### 3. Downloading an ISO Image

To create a VM, an ISO image is required. The following lines downloads the latest Ubuntu ISO from the official Ubuntu releases. **DatastoreId** and **NodeName** to specify where the ISO will be stored on the Proxmox node, **Url** pointing to the download source.

```go
iso, err := download.NewFile(ctx, "latest-ubuntu22-jammy-iso", &download.FileArgs{
    ContentType: pulumi.String("iso"),
    DatastoreId: pulumi.String("local"),
    NodeName:    pulumi.String("pve"),
    Url:         pulumi.String("https://releases.ubuntu.com/jammy/ubuntu-22.04.5-live-server-amd64.iso"),
}, pulumi.Provider(provider))
```

### 4. Creating the Virtual Machine

The following code defines a new VM configuration and provisions it.

```go
newVM, err := vm.NewVirtualMachine(ctx, "vm", &vm.VirtualMachineArgs{
    NodeName: pulumi.String("pve"),
    Name:     pulumi.String("vm"),
    Cpu: &vm.VirtualMachineCpuArgs{
        Cores: pulumi.Int(2),
    },
    Memory: &vm.VirtualMachineMemoryArgs{
        Dedicated: pulumi.Int(4800),
        Floating:  pulumi.Int(4800),
    },
    Disks: &vm.VirtualMachineDiskArray{
        vm.VirtualMachineDiskArgs{
            Size:       pulumi.Int(25),
            Interface:  pulumi.String("scsi0"),
            Iothread:   pulumi.Bool(true),
            FileFormat: pulumi.String("raw"),
        },
    },
    Cdrom: vm.VirtualMachineCdromArgs{
        Enabled:   pulumi.Bool(true),
        FileId:    iso.ID(),
        Interface: pulumi.String("ide3"),
    },
    BootOrders: pulumi.StringArray{
        pulumi.String("scsi0"),
        pulumi.String("ide3"),
        pulumi.String("net0"),
    },
    ScsiHardware:    pulumi.String("virtio-scsi-single"),
    OperatingSystem: vm.VirtualMachineOperatingSystemArgs{Type: pulumi.String("l26")},
    NetworkDevices: vm.VirtualMachineNetworkDeviceArray{
        vm.VirtualMachineNetworkDeviceArgs{
            Model:  pulumi.String("virtio"),
            Bridge: pulumi.String("vmbr0"),
        },
    },
    OnBoot: pulumi.Bool(true),
}, pulumi.DependsOn([]pulumi.Resource{iso}), pulumi.Provider(provider))
```

* `Cpu`: Allocates 2 CPU cores to the VM.

* `Memory`: Assigns 4800 MB (4.8 GB) of memory to both dedicated and floating memory allocations.

* `Disks`: Creates a 25 GB disk with the SCSI interface, using the raw file format and enabling I/O threading for better performance.

* `Cdrom`: Configures a virtual CD-ROM to boot from the ISO file previously downloaded. It's attached to the ide3 interface.

* `BootOrders`: Specifies the sequence in which devices are checked during the boot process—first the SCSI disk ("scsi0"), then the CD-ROM ("ide3"), followed by the network adapter ("net0").

* `ScsiHardware`: Sets the SCSI controller to "[irtio-scsi-single](https://forum.proxmox.com/threads/virtio-scsi-vs-virtio-scsi-single.28426/)" for better I/O performance.

* `OperatingSystem`: Specifies the OS type as Linux 2.6 ("l26"), which is appropriate for modern Linux distributions.

* `NetworkDevices`: Adds a network adapter using the "virtio" model for optimal performance and attaches it to the "vmbr0" bridge.

* `OnBoot`: Ensures the VM starts automatically when the Proxmox node boots.

* `DependsOn and Provider`: Ensures that the VM creation depends on the ISO file download being complete and uses the specified Proxmox provider.

<div align="center">* * * *</div>

<center>

Originally published on [Medium](https://medium.com/@madhankumaravelu93/building-a-home-lab-server-part-ii-45c7273332a8)

🌟 🌟 🌟 **The source code for this blog post can be found here** 🌟🌟🌟

[GitHub - madhank93/pulumi-proxmox at proxmox_vm](https://github.com/madhank93/pulumi-proxmox/tree/proxmox_vm)

</center>

**Reference**:

[1] [https://pve.proxmox.com/pve-docs/chapter-qm.html](https://pve.proxmox.com/pve-docs/chapter-qm.html)

[2] [https://pve.proxmox.com/pve-docs/qm.conf.5.html](https://pve.proxmox.com/pve-docs/qm.conf.5.html)

[3] [https://www.pulumi.com/registry/packages/proxmoxve/](https://www.pulumi.com/registry/packages/proxmoxve/)

[4] [https://github.com/bpg/terraform-provider-proxmox](https://github.com/bpg/terraform-provider-proxmox)
