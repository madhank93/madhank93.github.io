+++
title = "Building pre-configured VM images with Packer and GitHub actions"
description = "Simplifying VM provisioning and accelerating boot times"
date = 2025-09-01T00:00:00+00:00

[taxonomies]
tags = ["packer", "github-actions", "hashicorp", "qemu", "kvm", "vm image"]

[extra]
toc = true
+++

![Banner](https://cdn-images-1.medium.com/max/3840/1*AsrF1Sxe-3gOgpMsjmc4Yw.png)

I recently hit a roadblock in my homelab setup when trying to install few packages as part of cloud-init during VM creation. Main issue I faced were the QEMU guest agent not running properly, which made getting the IP addresses of newly spun VMs difficult. That’s when I came across [**Hashicorp Packer**](https://developer.hashicorp.com/packer) — a tool that allows to pre-build cloud images with all the necessary packages already installed, and eliminating lengthy VM boot.

[**QEMU**](https://www.qemu.org/) (Quick Emulator) — An open-source hypervisor that emulates physical computers by providing virtualized hardware resources like storage, network cards, and CPU to guest operating systems.

[**KVM**](https://en.wikipedia.org/wiki/Kernel-based_Virtual_Machine) (Kernel-based Virtual Machine) — A Linux virtualization technology that turns the kernel into a hypervisor, allowing multiple isolated virtual machines to run on a single host with near-native performance.

[**Packer QEMU**](https://developer.hashicorp.com/packer/integrations/hashicorp/qemu/latest/components/builder/qemu) builder — Creates KVM virtual machine images by spinning up a temporary VM, installing an operating system, provisioning software, and then converting the result into a reusable image.

## Packer configuration breakdown

### Plugin and variable setup

```ini
packer {
    required_plugins {
        qemu = {
            version = ">= 1.1.4"
            source  = "github.com/hashicorp/qemu"
        }
    }
}
variable "image_url" {
    type    = string
    default = "https://cloud-images.ubuntu.com/releases/24.04/release/ubuntu-24.04-server-cloudimg-amd64.img"
}
variable "image_checksum" {
    type    = string
    default = "sha256:834af9cd766d1fd86eca156db7dff34c3713fbbc7f5507a3269be2a72d2d1820"
}
```

This section declares the required QEMU plugin and defines variables for the Ubuntu 24.04 cloud image URL and its SHA256 checksum for integrity verification.

### QEMU source configuration

```ini
source "qemu" "ubuntu" {
    iso_url              = var.image_url
    iso_checksum         = var.image_checksum
    disk_image           = true
    output_directory     = "artifacts"
    disk_interface       = "virtio"
    net_device           = "virtio-net"
    disk_size            = "8G"
    format               = "qcow2"
    disk_compression     = true
    accelerator          = "kvm"
    headless             = true
    qemuargs = [
        ["-cdrom", "cidata.iso"]
    ]
    ssh_username         = "ubuntu"
    ssh_password         = "supersecret"
    ssh_timeout          = "10m"
    shutdown_command     = "echo 'supersecret' | sudo -S shutdown -P now"
}
```

Key Configuration,

* **qcow2** — QEMU's copy-on-write format that supports compression and snapshots

* **accelerator** = "kvm" — Enables hardware acceleration for faster builds

* **qemuargs** — Attaches the cloud-init ISO containing user configuration

### Build process

```ini
build {
    sources = ["source.qemu.ubuntu"]
    provisioner "shell" {
        inline = [
            "while [ ! -f /var/lib/cloud/instance/boot-finished ]; do echo 'Waiting for Cloud-Init...'; sleep 1; done",
        ]
    }
    provisioner "shell" {
        inline = [
            "sudo apt-get clean",
            "sudo rm -rf /var/lib/apt/lists/*",
            "sudo fstrim -av"
        ]
    }
}
```

The build process,

 1. Waits for cloud-init completion to ensure all initial setup is finished

 2. Cleans package caches to reduce final image size

 3. Trims unused disk space for optimal storage efficiency

### Cloud-Init configuration

```yml
users:
    - name: ubuntu
    sudo: ALL=(ALL) NOPASSWD:ALL
    shell: /bin/bash
    lock_passwd: false
    plain_text_passwd: supersecret
ssh_pwauth: true
password: supersecret
chpasswd:
    expire: false
runcmd:
    - systemctl enable ssh
    - systemctl start ssh
    - systemctl restart ssh
package_update: true
package_upgrade: true
packages:
    - openssh-server
    - clang
    - llvm
    - make
    - gcc
    - git
    - curl
    - wget
    - net-tools
    - qemu-guest-agent
    - ubuntu-drivers-common
    - linux-headers-generic
write_files:
    - path: /etc/ssh/sshd_config.d/99-packer.conf
    content: |
        PasswordAuthentication yes
        PermitRootLogin no
        PubkeyAuthentication yes
    permissions: '0644'
```

This cloud-init configuration,

* Creates a user with sudo privileges and password authentication

* Installs essential development tools and system utilities

* Configures SSH for remote access

* Updates all packages to latest versions

## GitHub Actions automation

The GitHub Actions workflow automates the entire image building and release process.

### Workflow triggers and permissions

```yml
on:
    push:
    branches:
        - "main"
    workflow_dispatch:
permissions:
    contents: write
    id-token: write
```

Triggers on pushes to main branch or manual dispatch and grants permissions to create releases & upload artifacts.

### Environment Variables

```yml
env:
    PRODUCT_VERSION: "1.14.1"
    UBUNTU_VERSION: "22.04"
```

Defines Packer and Ubuntu version.

## GA step analysis

### 1. Repository checkout

```yml
- name: Checkout repository
    uses: actions/checkout@v4
```

Downloads the repository code.

### 2. KVM permissions setup

```yml
- name: Enable KVM group perms
    run: |
    echo 'KERNEL=="kvm", GROUP="kvm", MODE="0666", OPTIONS+="static_node=kvm"' | sudo tee /etc/udev/rules.d/99-kvm4all.rules
    sudo udevadm control --reload-rules
    sudo udevadm trigger --name-match=kvm
```

This step enables hardware acceleration for significantly faster builds.

### 3. QEMU installation

```yml
- name: Install QEMU and dependencies
    run: |
    sudo apt-get update
    sudo apt-get install -y qemu-system-x86 qemu-utils genisoimage
```

Installs QEMU system emulator, utilities, and tools for creating ISO images.

### 4. Cloud-Init ISO creation

```yml
- name: Create Cloud-Init ISO
    run: |
    genisoimage -output cidata.iso -input-charset utf-8 -volid cidata -joliet -r http/user-data http/meta-data
```

Creates an ISO containing cloud-init configuration files (user-data) that QEMU will mount as a CD-ROM during VM boot.

### 5. Packer setup and execution

```yml
- name: Setup Packer
    uses: hashicorp/setup-packer@main
    id: setup
    with:
    version: ${{ env.PRODUCT_VERSION }}

- name: Initialize Packer
    id: init
    run: packer init .

- name: Validate Packer template
    run: packer validate .

- name: Build artifact
    run: packer build .
```

Downloads Packer, initializes plugins, validates the template syntax, and builds the custom image.

### 6. Release creation

```yml
- name: Rename artifact for release
    run: |
    mv artifacts/* ubuntu-${{ github.ref_name }}.qcow2
- name: Create Release and Upload Artifact
    uses: softprops/action-gh-release@v1
    with:
    files: ubuntu-${{ github.ref_name }}.qcow2
    tag_name: v${{env.UBUNTU_VERSION}}-${{ github.run_number }}
    name: Ubuntu custom image v${{env.UBUNTU_VERSION}}-${{ github.run_number }}
    body: |
        ## Ubuntu Image Release
        Built from commit: ${{ github.sha }}
        Branch: ${{ github.ref_name }}
        Build date: ${{ github.event.head_commit.timestamp }}
```

Renames the built image with a descriptive filename and creates a GitHub release with,

* Automatic version tagging using Ubuntu version and build number

* Detailed release notes including commit info and build timestamp

* The custom image file as a downloadable asset

<div align="center">* * * *</div>

<center>

Originally published on [Medium](https://medium.com/@madhankumaravelu93/building-pre-configured-vm-images-with-packer-and-github-actions-2847cadcbfa9)

🌟 🌟 🌟 **The source code for this blog post can be found here** 🌟🌟🌟

[GitHub - madhank93/ubuntu-cloud-image](https://github.com/madhank93/ubuntu-cloud-image)
</center>

**Reference:**

[1] [https://www.redhat.com/en/topics/virtualization/what-is-KVM](https://www.redhat.com/en/topics/virtualization/what-is-KVM)

[2] [https://www.qemu.org/documentation/](https://www.qemu.org/documentation/)

[3] [https://developer.hashicorp.com/packer](https://developer.hashicorp.com/packer)

[4] [https://actuated.com/blog/automate-packer-qemu-image-builds](https://actuated.com/blog/automate-packer-qemu-image-builds)
