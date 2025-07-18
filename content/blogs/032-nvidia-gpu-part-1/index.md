+++
title = "Making my homelab server AI ready with Nvidia GPU — Part I"
description = "A step-by-step upgrade from server to AI workstation"
date = 2025-06-25T00:00:00+00:00

[taxonomies]
tags = ["gpu", "homelab", "ai", "nvidia", "egpu"]

[extra]
toc = true
+++

![Icons - flaticon](https://cdn-images-1.medium.com/max/3840/1*sYyFSKq0gUUmN7V2K8Jj4g.png)

When I set out to upgrade my refurbished Dell Precision 7820 tower for AI workloads, I needed serious GPU computing power. There are two main types: consumer GPUs and enterprise GPUs. Consumer GPUs are usually cheaper and good enough for most personal or small business needs, while enterprise GPUs are more reliable and powerful, but also much more expensive.

The Nvidia GEFORCE RTX 5070 Ti was my card of choice, but fitting it into my server wasn’t as simple as dropping it into a PCIe slot.

![Nvidia RTX 5070-Ti](https://cdn-images-1.medium.com/max/4000/0*xiMGncmNwrDR--4b.com/StaticFile/Image/Global/b456806454c82f2cf286d7c95a892515/Product/43913/Png)

* The card is huge — it barely fits inside the case.

* It gets really hot, and worried about overheating the whole server.

* I wanted flexibility — sometimes I’d like to use the GPU with my laptop, not just the server. Swapping it in and out of the case would be a pain.

## eGPU Dock

Instead of cramming the GPU inside the server, I decided to use an external GPU dock (Apple’s Mac M series doesn’t support). After some research, I went with the [AOOSTAR AG02 eGPU dock](https://aoostar.com/products/aoostar-ag01-egpu-dock-with-oculink-port-built-in-huntkey-400w-power-supply-supports-tgx-interface-hot-swap?variant=49246191223082).

![AOOSTAR — AG02](https://cdn-images-1.medium.com/max/2000/0*xqUY5lziseghMEbs)

* It supports both OCuLink and USB4/Thunderbolt connections, which are super fast.

* It has a strong built-in power supply, so I didn’t have to worry about powering the GPU.

* Its open design means even big cards fit and stay cool.

## Adding thunderbolt

![Thunderbolt](https://cdn-images-1.medium.com/max/2000/0*HJ1sORadBiScO3XV.jpg)

My Dell 7820 server didn’t have Thunderbolt ports out of the box. To fix this, I bought a Dell P1XY1 Thunderbolt 3 PCIe Expansion Card. This card plugs into the server and adds Thunderbolt capability, which is needed for high-speed external GPU use.

## Important BIOS settings

Once the hardware was ready, I had to tweak some settings in the BIOS.

* Turn on Thunderbolt support.

* Set Thunderbolt security to “User Authorization” (or “No Security” if you want fewer connection prompts).

* Set PCIe Bus Allocation to “Optimize for Thunderbolt.”

## Right cable

At first, I tried using a regular USB-C cable to connect the dock. It didn’t work well — the GPU kept disconnecting or wasn’t detected at all. Turns out, you must use a Thunderbolt-certified cable for this setup. Once I switched to the Thunderbolt cable that came with the dock, everything worked perfectly.

![GPU detection](https://cdn-images-1.medium.com/max/3816/1*y5j61AprMLy3QS8NqGFBmQ.png)

A vGPU, or virtual GPU, is a technology that allows a physical GPU to be split into virtual instances, enabling multiple virtual machines (VMs) to share graphics card. However, this feature is officially supported only on Nvidia’s enterprise-grade GPUs and is not available for consumer models like the RTX series. For homelab users, the alternative is PCIe tunneling (passthrough), which assigns the entire GPU to a single VM at a time — without sharing it across multiple VM.

<div align="center">* * * *</div>

<center>

Originally published on [Medium](https://medium.com/@madhankumaravelu93/making-my-homelab-server-ai-ready-with-nvidia-gpu-part-i-e292ad0ea042)

</center>

### Reference:

[1] [https://aoostar.com/collections/egpu-series](https://aoostar.com/collections/egpu-series)

[2] [https://www.nvidia.com/en-us/geforce/graphics-cards/50-series/](https://www.nvidia.com/en-us/geforce/graphics-cards/50-series/)
