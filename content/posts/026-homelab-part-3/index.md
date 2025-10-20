+++
title = "Building a Home Lab server — Part III"
description = "Automating LXC container deployment using Pulumi and Go in Proxmox"
date = 2024-11-03T00:00:00+00:00

[taxonomies]
tags = ["homelab", "proxmox", "self-hosting", "containers", "lxc", "pulumi", "golang"]

[extra]
toc = true
+++

![](https://cdn-images-1.medium.com/max/3840/1*klPR_aiCnV51Qh6AWOkLNg.jpeg)

In this article we will explore how to automate the process of [LXC](https://linuxcontainers.org/lxc/introduction/) container deployment on [Proxmox](https://www.proxmox.com/en/) using [Pulumi](https://www.pulumi.com/).

## How LXC is different from Docker ?

_**Application containers (like Docker)**_ run a single process and are designed for stateless, ephemeral workloads, allowing you to quickly scale up or down without worrying about container lifecycle. In contrast, _**System containers (like LXC)**_ act more like virtual machines, running a full OS where you can manage services, install packages, and apply updates. They’re long-lasting and managed like traditional servers, receiving regular security updates directly from the OS distribution.

## Code Walkthrough

### 1. Loading Environment Variables

The script starts by loading environment variables from a `.env` file using the [godotenv](https://github.com/joho/godotenv) package.

`.env` file

```ini
PROXMOX_USERNAME=XXXXXXXXXXXX
PROXMOX_PASSWORD=XXXXXXXXXXXX
CONTAINER_PASSWORD=XXXXXXXXXX
```

`main.go` file

```go
err := godotenv.Load()
```

It then reads the Proxmox username, password and container password from the environment. These values are used to authenticate with the Proxmox server.

```go
proxmoxUsername := os.Getenv("PROXMOX_USERNAME")
proxmoxPassword := os.Getenv("PROXMOX_PASSWORD")
containerPassword := os.Getenv("CONTAINER_PASSWORD")

if proxmoxUsername == "" || proxmoxPassword == "" || containerPassword == "" {
    return fmt.Errorf("PROXMOX_USERNAME, PROXMOX_PASSWORD and CONTAINER_PASSWORD must be set in the .env file")
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

### 3. Download LXC template

The below code downloads an LXC template for Ubuntu 22.04 from a public image repository using the **download.NewFile** function.

```go
ctTemplate, err := download.NewFile(ctx, "latest-ubuntu22-jammy-lxc", &download.FileArgs{
    ContentType: pulumi.String("vztmpl"),
    DatastoreId: pulumi.String("local"),
    NodeName:    pulumi.String("pve"),
    Url:         pulumi.String("https://images.linuxcontainers.org/images/ubuntu/jammy/amd64/default/20241101_07%3A42/rootfs.tar.xz"),
}, pulumi.Provider(provider))
```

### 4. Creating the Container

Once the template is successfully downloaded, it then proceeds to create a new LXC container in Proxmox.

```go
newCT, err := ct.NewContainer(ctx, "container", &ct.ContainerArgs{
    NodeName: pulumi.String("pve"),
    OperatingSystem: &ct.ContainerOperatingSystemArgs{
        TemplateFileId: ctTemplate.ID(),
        Type:           pulumi.String("ubuntu"),
    },
    Memory: &ct.ContainerMemoryArgs{
        Dedicated: pulumi.Int(14096),
        Swap:      pulumi.Int(4096),
    },
    Cpu: &ct.ContainerCpuArgs{
        Cores: pulumi.Int(2),
    },
    Disk: &ct.ContainerDiskArgs{
        DatastoreId: pulumi.String("local-lvm"),
        Size:        pulumi.Int(10),
    },
    NetworkInterfaces: ct.ContainerNetworkInterfaceArray{
        &ct.ContainerNetworkInterfaceArgs{
            Name:    pulumi.String("eth0"),
            Bridge:  pulumi.String("vmbr0"),
            Enabled: pulumi.Bool(true),
        },
    },
    Initialization: &ct.ContainerInitializationArgs{
        UserAccount: &ct.ContainerInitializationUserAccountArgs{
            Password: pulumi.String(containerPassword),
        },
    },
    VmId:    pulumi.Int(200),
    Started: pulumi.Bool(true),
}, pulumi.DependsOn([]pulumi.Resource{ctTemplate}), pulumi.Provider(provider))
```

<div align="center">* * * *</div>

<center>

Originally published on [Medium](https://medium.com/@madhankumaravelu93/building-a-home-lab-server-part-iii-65844f344ac2)

🌟 🌟 🌟 **The source code for this blog post can be found here** 🌟🌟🌟

[GitHub - madhank93/pulumi-proxmox at proxmox_container](https://github.com/madhank93/pulumi-proxmox/tree/proxmox_container)

</center>

**Reference**:

[1] [https://ubuntu.com/blog/lxd-vs-docker](https://ubuntu.com/blog/lxd-vs-docker)

[2] [https://ubuntu.com/blog/what-are-linux-containers](https://ubuntu.com/blog/what-are-linux-containers)

[3] [https://www.pulumi.com/registry/packages/proxmoxve/api-docs/ct/container/](https://www.pulumi.com/registry/packages/proxmoxve/api-docs/ct/container/)

[4] [https://github.com/bpg/terraform-provider-proxmox](https://github.com/bpg/terraform-provider-proxmox)

[5] [https://images.lxd.canonical.com/](https://images.lxd.canonical.com/)
