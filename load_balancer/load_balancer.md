## Load Balancer

A **load balancer** is a system that automatically distributes incoming network traffic across multiple backend servers or services. Its main purpose is to ensure:

1. **High availability** → If one server goes down, traffic is redirected to healthy ones.
2. **Scalability** → Spreads workload so no single server is overwhelmed.
3. **Performance** → Improves response times by balancing requests.
4. **Reliability** → Provides redundancy and fault tolerance.

## Kinds

- **kube-proxy** = in-cluster Pod load balancing (basic, L4 only).
    - iptables mode: Round-robin
- **Cloud/external Load Balancer** = external traffic entry + advanced load balancing + HA.

## **How it works**

- Clients send requests to a single load balancer endpoint (IP/DNS).
- The load balancer forwards each request to one of the backend servers using a chosen algorithm.

## **AWS** ALB/NLB/CLB

### 1. **Application Load Balancer (ALB)**

- Layer 7
- **Round Robin (default)** → distributes new connections evenly across healthy targets.
- **Least Outstanding Requests** → directs new requests to the target with the fewest active requests (good for uneven workloads).
- **Weighted Target Groups** → you can shift traffic between groups (e.g., Canary releases, A/B testing).

### 2. **Network Load Balancer (NLB)**

- Layer 4
- **Flow Hash algorithm** → balances TCP/UDP connections based on a hash of:
    - Protocol, Source IP, Source Port, Destination IP, Destination Port.
- This provides **stickiness at the connection level** (same client flow → same target).
- Very fast, designed for millions of requests per second.

### 3. **Classic Load Balancer (CLB)** (legacy, not recommended for new apps)

- Layer 4/7
- **Round Robin** for TCP/HTTP.
- **Sticky Sessions** available (via cookies).

## L4 LB vs. L7 LB

| Feature | **L4 Load Balancer** | **L7 Load Balancer** |
| --- | --- | --- |
| OSI Layer | Layer 4 (Transport: TCP/UDP) | Layer 7 (Application: HTTP/HTTPS/gRPC) |
| Decision Basis | IP, Port, Protocol | URL path, headers, cookies, method |
| Performance | Very fast, low overhead | Slightly slower (parses app data) |
| Flexibility | Simple connection distribution | Advanced routing + content inspection |
| Examples | AWS NLB, IPVS, HAProxy (TCP) | AWS ALB, NGINX Ingress, Traefik |
| Best For | Databases, gaming, raw TCP/UDP | APIs, websites, microservices |

## **External Load Balancer Strategies**

### 1. **Round Robin**

- Sends requests to servers in sequence (A → B → C → A …).
- **Fair** if servers are equal, but doesn’t account for load.

### 2. **Least Connections**

- Sends the request to the server with the **fewest active connections**.
- Good when requests have variable lengths (some users keep long connections open).

### 3. **Least Response Time**

- Sends traffic to the server with the **lowest response time** (latency).
- Often combined with **least connections**.

### 4. **Weighted Round Robin**

- Like round robin, but some servers get **more requests** if they have higher weight (capacity).
- Example: Server A (weight 2), B (weight 1) → A gets 2 requests for every 1 to B.

### 5. **Weighted Least Connections**

- Combines weights and least-connections → high-capacity servers get more traffic, but still balances based on active connections.

### 6. **IP Hash / Source Hash**

- Uses client IP hash to pick the server.
- Ensures **session stickiness** (same client always hits the same server).

### 7. **Random with Weight**

- Picks a server randomly, but weighted probabilities.
- Example: A (70%), B (20%), C (10%).

### 8. **Geolocation-based (GeoDNS / GSLB)**

- Directs traffic based on the client’s **geographic location**.
- Example: Users in US go to US servers, users in EU go to EU servers.

### 9. **Consistent Hashing**

- Used in CDNs and caches.
- Ensures the same request key (like a file or session ID) always goes to the same server.