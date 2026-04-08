+++
title = "System Design 101: Load Balancing"
description = "A hands-on guide to implementing and testing load balancing algorithms using Golang, Nginx, and Docker"
date = 2025-10-26T00:00:00+00:00

[taxonomies]
tags = ["system-design", "golang", "nginx", "docker", "load-balancing"]

[extra]
toc = true
+++

![Banner](https://cdn-images-1.medium.com/max/3840/1*banner-load-balancing.png)

Load balancing is the process of distributing incoming network traffic across multiple servers to improve reliability, scalability, and performance. The practice prevents individual servers from becoming overwhelmed and enables systems to handle increased demand efficiently.

## Operating Levels

Load balancing operates at distinct network stack layers:

**Layer 4 (Transport Level)** — Routes traffic using TCP/UDP protocols based on IP addresses and ports. Common algorithms: Round Robin, Least Connections.

**Layer 7 (Application Level)** — Examines HTTP headers, cookies, and request content to determine routing. Examples: Geo-based routing, Consistent Hashing.

## Common Algorithms

### Round Robin

Distributes requests sequentially across all available servers in a circular order. Simple and effective when all servers have similar capacity and workload.

### Least Connections

Routes each new request to the server with the fewest active connections. Better suited for workloads with variable processing time per request.

### Geo-based Routing

Routes traffic based on the geographic location of the client. Minimises latency by directing users to the nearest data centre or server.

### Consistent Hashing

Maps both requests and servers onto a hash ring. A request is routed to the nearest server on the ring — minimising redistribution when servers are added or removed.

## Practical Implementation

This article demonstrates building a basic REST API server using Go alongside an Nginx load balancer to evaluate Round Robin, Least Connections, Geo-based routing, and Consistent Hashing strategies.

### Application Server (Go)

The Go server reads its identity and region from environment variables, making it easy to spin up multiple instances with distinct configurations:

```go
package main

import (
    "fmt"
    "net/http"
    "os"
)

func main() {
    serverID := os.Getenv("SERVER_ID")
    region   := os.Getenv("REGION")

    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "Response from server %s (region: %s)\n", serverID, region)
    })

    http.ListenAndServe(":8080", nil)
}
```

### Nginx — Round Robin

```nginx
upstream backend {
    server app1:8080;
    server app2:8080;
    server app3:8080;
}

server {
    listen 80;
    location / {
        proxy_pass http://backend;
    }
}
```

### Nginx — Least Connections

```nginx
upstream backend {
    least_conn;
    server app1:8080;
    server app2:8080;
    server app3:8080;
}
```

### Docker Compose

```yaml
services:
  app1:
    build: .
    environment:
      - SERVER_ID=1
      - REGION=us-east

  app2:
    build: .
    environment:
      - SERVER_ID=2
      - REGION=us-west

  app3:
    build: .
    environment:
      - SERVER_ID=3
      - REGION=eu-west

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - app1
      - app2
      - app3
```

## Testing

Run the stack with Docker Compose and send repeated requests to observe how traffic is distributed:

```bash
docker compose up -d

for i in $(seq 1 9); do
  curl -s http://localhost/
done
```

With Round Robin you'll see requests cycling across all three servers. With Least Connections, the distribution adapts to active load.

<div align="center">* * * *</div>

<center>

Originally published on [Medium](https://medium.com/@madhankumaravelu93/system-design-101-load-balancing-87483da7be93)

🌟 🌟 🌟 **The source code for this blog post can be found here** 🌟🌟🌟

[GitHub - madhank93/learn-system-design](https://github.com/madhank93/learn-system-design)

</center>

## References

[1] [https://nginx.org/en/docs/http/load_balancing.html](https://nginx.org/en/docs/http/load_balancing.html)

[2] [https://www.getzola.org/](https://www.getzola.org/)

[3] [https://go.dev/doc/](https://go.dev/doc/)
