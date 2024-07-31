+++
title = "Unlocking geo-restricted contents with Tailscale"
description = "Getting started with Mesh VPN using Tailscale"
date = 2024-06-08T00:00:00+00:00

[taxonomies]
tags = ["tailscale", "networking", "vpn"]

[extra]
toc = true
+++

![](https://cdn-images-1.medium.com/max/3840/1*B-M-HCcY7njKreEwVttZtg.png)

In this blog post, we will look at how to access geo-restricted content using Tailscale’s Mesh VPN concepts.

### What is Tailscale?

Tailscale is a modern VPN solution built on top of the _***[WireGuard](https://www.wireguard.com/)***_ protocol that simplifies creating and managing secure network access between devices. It uses a _***[mesh networking](https://tailscale.com/learn/understanding-mesh-vpns)***_ approach, enabling encrypted peer-to-peer connections, each device can connect directly to other devices within the network, without needing a central server to route traffic.

### How does Tailscale Mesh VPN work?

![](https://cdn-images-1.medium.com/max/4000/0*Wv05k3paq7yZRjrv)

![1. Traditional VPN and 2. Tailscale Mesh VPN](https://cdn-images-1.medium.com/max/4000/0*7d2fGljfL270xLq8)

A traditional VPN creates a secure, encrypted tunnel between your device and a centralized VPN server hub. This tunnel allows your internet traffic to travel securely through the server, masking your IP address and encrypting your data.

![Traditional VPN flow](https://cdn-images-1.medium.com/max/2000/1*nESiyXE4AcsKm36oenEHRw.png)

Tailscale uses a mesh VPN architecture, allowing direct, peer-to-peer connections between devices without a central VPN server. The Tailscale coordination server is used for authentication, key exchange, and device discovery. Once devices are authenticated and have discovered each other, they establish direct, peer-to-peer connections using Wireguard.

![Tailscale VPN flow](https://cdn-images-1.medium.com/max/2000/1*S7AokFErYLmElnbb8Qcr8A.png)

Tailscale supports many devices (Windows, Mac, Linux, iOS, and Android), and the _***[installation](https://tailscale.com/kb/1347/installation)***_ process is pretty straightforward.

### So how does it allow accessing geo-restricted content?

![](https://cdn-images-1.medium.com/max/2000/1*LKOTOu_faAUl-UoxOLKGtA.png)

![1. Exit node flow 2. Tailscale admin console](https://cdn-images-1.medium.com/max/2930/1*dHpzjgrDiZ32K7c_Ef6g1Q.png)

One of the many features of Tailscale is ***[Exit nodes](https://tailscale.com/kb/1103/exit-nodes)***, an exit node is a device within the Tailscale network that routes traffic to the broader internet on behalf of other devices in the network. This is useful for scenarios where you want to use a specific network’s internet connection, such as accessing region-specific content or securing internet traffic through a trusted network.

**References**:

[1] [https://tailscale.com/blog/how-tailscale-works](https://tailscale.com/blog/how-tailscale-works)

[2] [https://tailscale.com/learn/understanding-mesh-vpns](https://tailscale.com/learn/understanding-mesh-vpns)

[3] [https://tailscale.com/blog/how-nat-traversal-works](https://tailscale.com/blog/how-nat-traversal-works)

[4] [https://tailscale.com/kb/1103/exit-nodes](https://tailscale.com/kb/1103/exit-nodes)
