+++
title = "Serving LLMs from Raspberry Pi using Ollama and Cloudflare Tunnel"
description = "Provisioning the compute resource for k8s setup in AWS using Pulumi"
date = 2024-04-21T01:25:33+05:30

[taxonomies]
tags = ["llm", "raspberry-pi", "cloudflare-tunnel", "ollama", "ansible", "docker", "self-hosting"]

[extra]
toc = true
+++

![](https://cdn-images-1.medium.com/max/3840/0*HDUywKYg7M4Ij64B)

In this article, we will look at how to run LLMs like Gemma and Llama 2 using Ollama on the Raspberry Pi and using [Cloudflare tunnels](https://www.cloudflare.com/products/tunnel/) for public internet access.

- [Gemma](https://github.com/google-deepmind/gemma) and [Llama2](https://github.com/meta-llama/llama) are the open-source LLMs released by Google and Facebook.

- [Ollama](https://ollama.com/) is an open-source project that makes it easier to create and run a LLM model locally.

- [Raspberry Pi](https://www.raspberrypi.com/) is a single-board computer powered by [ARM](https://www.redhat.com/en/topics/linux/what-is-arm-processor) processors.

- [Open WebUI](https://github.com/open-webui/open-webui) provides a user-friendly Web UI for LLMs.

- [Cloudflare tunnels](https://www.cloudflare.com/products/tunnel/) connect LLMs running locally on Raspberry Pi to the internet.

![High-level architecture](https://cdn-images-1.medium.com/max/2252/1*OFpYLfvtbnXUIcjhih3r0Q.png)

## Installing Ollama and Open Web UI

Using the following docker-compose file to install the Ollama and Open Web UI as containers on the Raspberry Pi.

{{ gist(url="https://gist.github.com/madhank93/8f493338dc792af68595f411d7fc2dcb", class="gist") }}

Ansible scripts are used to install prerequisites setup and above docker containers on the Raspberry Pi.

```yml
- name: Install Ollama in Raspberry Pi
    become: true
    block:
    - name: Git checkout Open webui (ollama webui)
        ansible.builtin.git:
        repo: "https://github.com/open-webui/open-webui.git"
        dest: "/home/{{ user }}/open-webui"
        version: "{{ open_webui.git_tag }}"
    - name: Copy docker compose file
        ansible.builtin.template:
        mode: u=rwx,g=rx,o=rx
        src: "{{ playbook_dir }}/roles/ollama/templates/docker-compose.yml.j2"
        dest: "/home/{{ user }}/open-webui/docker-compose.yml"
    - name: Run Docker Compose
        community.docker.docker_compose:
        debug: true
        project_src: "/home/{{ user }}/open-webui"
        files:
            - docker-compose.yml
        state: present
```

Once the Ansible script is executed successfully, Open Web UI can be accessed at ip_addr_of_raspberrypi:3000 . Login into the application as an Admin and head onto the Settings > Model page to download the Gemma and Lama 2 models.

## Cloudflare tunnel setup

Cloudflare tunnels allow you to securely connect Raspberry Pi to the internet.

### i. Install and configure Cloudflared

```sh
# Download the package
curl -L https://github.com/cloudflare/cloudflared/releases/download/2024.2.1/cloudflared-linux-arm64.deb

# Install the package
sudo dpkg -i cloudflared.deb

# Authenticate cloudflared setup
cloudflared tunnel login

# Create Cloudflare tunnel
cloudflared tunnel create ollama-ai
```

### ii. Create a configuration file

```yml
url: http://localhost:3000
tunnel: <Tunnel-UUID>
credentials-file: /root/.cloudflared/<Tunnel-UUID>.json
```

### iii. Start routing traffic

```sh
# Routing the traffic
cloudflared tunnel route dns ollama-ai ai.madhan.app

# Run the tunnel
cloudflared tunnel run ollama-ai
```

When the Cloudflare tunnel setup is completed successfully Gemma and Lama2 models can be accessed using Open Web UI at ai.madhan.app (the site is no longer operational)

<div align="center">* * * *</div>

<center>

Originally published on [Medium](https://medium.com/@madhankumaravelu93/serving-llms-from-raspberry-pi-using-ollama-and-cloudflare-tunnel-a688930583cc)

🌟 🌟 🌟 **The source code for this blog post can be found here** 🌟🌟🌟

[GitHub - madhank93/ollama-pi-cf](https://github.com/madhank93/ollama-pi-cf)

</center>

**References:**

[1] [https://ollama.com/blog](https://ollama.com/blog)

[2] [https://docs.openwebui.com/](https://docs.openwebui.com/)

[3] [https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/)
