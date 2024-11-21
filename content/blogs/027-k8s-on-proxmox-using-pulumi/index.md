+++
title = "Setting up Kubernetes on Proxmox with Cloud-Init and Pulumi— Part I"
description = "Provisioning the compute resource for k8s"
date = 2024-11-17T00:00:00+00:00

[taxonomies]
tags = ["homelab", "proxmox", "self-hosting", "k8s", "cloud-init", "pulumi", "golang"]

[extra]
toc = true
+++

![Banner](https://cdn-images-1.medium.com/max/3840/1*O8D_62j7_OCIe4dstOwnvg.jpeg)

In this blog post, we’ll explore how to set up a [Kubernetes](https://kubernetes.io/) cluster on [Proxmox](https://www.proxmox.com/en/) using [cloud-init](https://cloud-init.io/) for VM initialization during their first boot and [Pulumi](https://www.pulumi.com/) for infrastructure as code.

## Prerequisite

![Enable snippets in Proxmox](https://cdn-images-1.medium.com/max/6028/1*d36gWJKGtwTG3f04SHFUsw.png)

**Ensure snippets are enabled in Storage Configuration:**

* Go to **Datacenter > Storage** in the Proxmox GUI.

* Edit an existing storage (or add a new one) that supports snippets

* Ensure the `Content` field includes the **Snippets** type, as showed in the above screenshot.

## Cloud-Init Configuration

Let’s break down the cloud-init configuration used to set up k8s nodes.

### User Setup

```yaml
users:
    - name: ubuntu
    sudo: ALL=(ALL) NOPASSWD:ALL
    lock_passwd: false
    passwd: $6$SP/vqykLkV9d05An$mJ/fEZ3gmfVvD1vwSJqxfjsK9z/bykIMbCZ/Hov.nt31e8h0XklDSE7ofw2YjPemVOSm14JdYoEfEzbxkFkY/1
    ssh_authorized_keys:
        - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHDtJdQ12Q8pUUGM16V1Ko+es5LzuGT/0FGWWTmsKQxj
```

This section creates a user named “ubuntu” with sudo privileges and adds SSH access.

### System Configuration

```yaml
write_files:
    - path: /etc/modules-load.d/k8s.conf
    permissions: '0644'
    content: |
        overlay
        br_netfilter
    - path: /etc/sysctl.d/k8s.conf
    permissions: '0644'
    content: |
        net.bridge.bridge-nf-call-iptables  = 1
        net.bridge.bridge-nf-call-ip6tables = 1
        net.ipv4.ip_forward                 = 1
```

These configurations includes necessary kernel modules and system parameters for Kubernetes

### Network Configuration

```yaml
- path: /etc/netplan/00-installer-config.yaml
    permissions: '0600'
    content: |
        network:
        version: 2
        ethernets:
            ens18:
            dhcp4: true
            dhcp6: false
            optional: true
            nameservers:
                addresses: [8.8.8.8, 8.8.4.4]
```

This sets up the network interface using Netplan, configuring DHCP and DNS

### Installation and Configuration Commands

```yaml
runcmd:
    # Set hostname
    - hostnamectl set-hostname ${hostname}

    # Basic system setup
    - modprobe overlay
    - modprobe br_netfilter
    - sysctl --system
    - systemctl restart systemd-timesyncd
    
    # Network configuration
    - netplan generate
    - netplan apply
    
    # Wait for network connectivity
    - |
    count=0
    max_attempts=30
    until ping -c 1 8.8.8.8 >/dev/null 2>&1 || [ $count -eq $max_attempts ]; do
        echo "Waiting for network connectivity... Attempt $count of $max_attempts"
        sleep 5
        count=$((count + 1))
    done
    
    # Update and install basic packages
    - apt-get update
    - apt-get install -y apt-transport-https ca-certificates curl software-properties-common
    
    # Setup Kubernetes repository
    - mkdir -p /etc/apt/keyrings
    - curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | gpg --dearmor -o /etc/apt/keyrings/kubernetes-archive-keyring.gpg
    - echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' > /etc/apt/sources.list.d/kubernetes.list
    
    # Install packages
    - apt-get update
    - |
    DEBIAN_FRONTEND=noninteractive apt-get install -y \
        wget \
        vim \
        net-tools \
        qemu-guest-agent \
        kubelet \
        kubeadm \
        kubectl \
        containerd
    
    # Hold kubernetes packages
    - apt-mark hold kubelet kubeadm
    
    # Install Nix package manager
    - curl --proto '=https' --tlsv1.2 -sSf -L https://install.determinate.systems/nix | sh -s -- install --no-confirm
    
    # Enable and start services
    - systemctl enable qemu-guest-agent
    - systemctl start qemu-guest-agent
    - systemctl enable kubelet
    - systemctl start kubelet

    # Set bash as the default shell
    - chsh -s /bin/bash ubuntu
 ```     

The runcmd section includes a series of commands to set the hostname, Load kernel modules, Apply system configurations, Set up networking, Install necessary packages including Kubernetes components and containerd, Install the Nix package manager and Enable and start required services.

## Pulumi Code for Kubernetes Setup

Now, let’s look at how we can use Pulumi with Go to set up our infra setup for k8s cluster on Proxmox.

### Cloud-init setup

Below function reads & customizes the cloud-init configuration and uploads the yml file into Proxmox.

```go
func createCloudInit(ctx *pulumi.Context, provider *proxmoxve.Provider, nodeName string, hostname string) (*storage.File, error) {
    configPath := filepath.Join("cloud-init", "cloud-init.yml")
    data, err := os.ReadFile(configPath)
    if err != nil {
        return nil, err
    }

    content := string(data)
    replacements := map[string]string{
        "${hostname}": hostname,
    }
    for k, v := range replacements {
        content = strings.ReplaceAll(content, k, v)
    }

    return storage.NewFile(ctx, hostname+"-cloud-init", &storage.FileArgs{
            NodeName:    pulumi.String(nodeName),
            DatastoreId: pulumi.String("local"),
            ContentType: pulumi.String("snippets"),
            FileMode:    pulumi.String("0755"),
            Overwrite:   pulumi.Bool(true),
            SourceRaw: storage.FileSourceRawArgs{
            Data:     pulumi.String(content),
            FileName: pulumi.String(hostname + ".yml"),
        },
    }, pulumi.Provider(provider))
}
```

### Provisioning VM

Below core function handles the creation of a VM.

```go
func createVM(ctx *pulumi.Context, provider *proxmoxve.Provider, config NodeConfig, cloudInit *storage.File, cloudImage *download.File, bridge string) (*vm.VirtualMachine, error) {
    return vm.NewVirtualMachine(ctx, config.Name, &vm.VirtualMachineArgs{
        NodeName: pulumi.String("pve"),
        Name:     pulumi.String(config.Name),
        Agent:    vm.VirtualMachineAgentArgs{Enabled: pulumi.Bool(true)},
        Cpu:      &vm.VirtualMachineCpuArgs{Cores: pulumi.Int(config.Cores)},
        Memory: &vm.VirtualMachineMemoryArgs{
            Dedicated: pulumi.Int(config.Memory),
            Floating:  pulumi.Int(config.Memory),
        },
        Disks: &vm.VirtualMachineDiskArray{
            vm.VirtualMachineDiskArgs{
                Size:       pulumi.Int(config.DiskSize),
                Interface:  pulumi.String("scsi0"),
                Iothread:   pulumi.Bool(true),
                FileFormat: pulumi.String("raw"),
                FileId:     cloudImage.ID(),
            },
        },
        BootOrders:   pulumi.StringArray{pulumi.String("scsi0"), pulumi.String("net0")},
        ScsiHardware: pulumi.String("virtio-scsi-single"),
        OperatingSystem: vm.VirtualMachineOperatingSystemArgs{
            Type: pulumi.String("l26"),
        },
        NetworkDevices: vm.VirtualMachineNetworkDeviceArray{
            vm.VirtualMachineNetworkDeviceArgs{
            Model:  pulumi.String("virtio"),
            Bridge: pulumi.String(bridge),
            },
        },
        OnBoot: pulumi.Bool(true),
        Initialization: &vm.VirtualMachineInitializationArgs{
            DatastoreId:    pulumi.String("local-lvm"),
            UserDataFileId: cloudInit.ID(),
        },
    }, pulumi.DependsOn([]pulumi.Resource{cloudImage}), pulumi.Provider(provider))
}
```

<div align="center">* * * *</div>

<center>

Originally published on [Medium](https://medium.com/@madhankumaravelu93/setting-up-kubernetes-on-proxmox-with-cloud-init-and-pulumi-part-i-64e697bdfe98)

🌟 🌟 🌟 **The source code for this blog post can be found here** 🌟🌟🌟

[GitHub - madhank93/pulumi-proxmox at main](https://github.com/madhank93/pulumi-proxmox/tree/main/proxmox-k8s)

</center>

**Reference**:

[1] [https://www.learnlinux.tv/how-to-build-an-awesome-kubernetes-cluster-using-proxmox-virtual-environment/](https://www.learnlinux.tv/how-to-build-an-awesome-kubernetes-cluster-using-proxmox-virtual-environment/)

[2] [https://www.youtube.com/watch?v=Kv6-_--y5CM](https://www.youtube.com/watch?v=Kv6-_--y5CM)

[3] [https://cloud-init.io/](https://cloud-init.io/)