+++
title = "Building a Home Lab server — Part I"
description = "Cost-effective approach to building a home lab server using refurbished workstation"
date = 2024-10-13T00:00:00+00:00

[taxonomies]
tags = ["homelab", "proxmox", "self-hosting", "servers"]

[extra]
toc = true
+++

![](https://cdn-images-1.medium.com/max/3840/1*RI95vE6PC0kzKZd7vHcm9A.png)

Setting up a home lab server is a great way to experiment with new technologies, learn about server management, and self-host diverse range of applications.

## What I have in my Homelab server ?

The refurbished [Dell Precision 7820](https://www.dell.com/en-us/shop/desktop-computers/precision-7820-tower-workstation/spd/precision-7820-workstation) workstation offers robust hardware, and 2.5Gbps Ethernet Switch, making it an excellent choice for a home lab server.

![Dell Precision 7820](https://cdn-images-1.medium.com/max/2000/0*XwoVzBq6WmJHKIxj)

* **Storage : 256GB NVMe SSD and 1TB HDD**

* **Memory : 48GB DDR4 ECC 2666MHz RAM**

* **CPU : Intel Xeon Gold 6134M (3.2 GHz, 8 cores, 16 threads)**

* **GPU : Radeon Pro WX5100 GPU**

## Choosing an Operating System (OS)

[Proxmox VE](https://www.proxmox.com/en/proxmox-virtual-environment/overview) — Proxmox is an open-source virtualization platform that supports [KVM-based virtualization](https://linux-kvm.org/page/Main_Page) and [LXC containers](https://linuxcontainers.org/). It provides a user-friendly web interface, making it easier to manage VMs and containers.

## Installing Proxmox VE

### **1. Download Proxmox VE ISO**

Visit the [Proxmox download](https://www.proxmox.com/en/downloads) page and download the latest Proxmox VE ISO.

### **2. Create a Bootable USB Drive**

Use tools like [Rufus](https://rufus.ie/en/) or [Etcher](https://etcher.balena.io/) to create a bootable USB drive with the downloaded ISO.

### **3. Boot from the USB Drive**

Insert the USB drive and boot the Dell Precision 7820. During startup, press the appropriate key (usually F12) to enter the **Boot Menu**, and select the USB drive.

### **4. Install Proxmox VE**

* Follow the installation prompts. Choose the **256GB NVMe SSD** as the installation target to ensure the OS takes advantage of the high-speed storage.

* Configure the network settings, hostname, and root password.

### **5. Initial Setup**

* After installation, you can access the **Proxmox web interface** by navigating to `https://<your-server-ip>:8006`.

* Login with the root credentials you created during installation.
