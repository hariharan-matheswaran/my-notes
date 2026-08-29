#aws

A **Load Balancer (LB)** acts as a reverse proxy that sits between clients and backend servers. It ==distributes incoming network or application traffic across multiple target instances== to ensure high availability, fault tolerance, and optimal performance.

### 1. Core Load Balancing Fundamentals

**Traffic Distribution Algorithms:**
- **Round Robin:** Sequential request distribution across available servers.
- **Least Connections:** Directs traffic to the server currently handling the fewest active connections.
- **IP Hash / Sticky Sessions (Session Affinity):** Directs requests from a specific client to the same server using client IP or cookies.

**Health Checks:** Periodic probes (`HTTP GET`, `TCP handshake`) to ==verify target health==. If a target fails checks, traffic is automatically diverted away until it recovers.

**SSL/TLS Termination:** Offloads cryptographic overhead by decrypting HTTPS traffic at the load balancer level before passing unencrypted (or re-encrypted) traffic to backend instances.

---

### 2. Layer 4 vs. Layer 7 Balancing

| Feature               | Layer 4 (Transport)                | Layer 7 (Application)                             |
| --------------------- | ---------------------------------- | ------------------------------------------------- |
| **OSI Layer**         | TCP / UDP                          | HTTP / HTTPS / gRPC / WebSockets                  |
| **Data Inspected**    | Source/Destination IP & Port       | URL Path, Host header, Cookies, HTTP Methods      |
| **Performance**       | Ultra-low latency, high throughput | Slightly higher latency due to packet inspection  |
| **Routing Decisions** | IP + Port mapping                  | Content-based routing (e.g., `/api` vs `/static`) |

---

### 3. AWS Elastic Load Balancing (ELB) Suite

AWS offers four types of managed load balancers under the **Elastic Load Balancing (ELB)** family:

CODE (Classic LB ignored in below Diagram)

```
                  ┌───────────────────────────────┐
                  │   Elastic Load Balancing      │
                  └───────────────┬───────────────┘
         ┌────────────────────────┼────────────────────────┬────────────────────────┐
         ▼                        ▼                        ▼                        ▼
┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐     
│       ALB        │     │       NLB        │     │       GWLB       │              
│   (Application)  │     │    (Network)     │     │    (Gateway)     │        
│     Layer 7      │     │     Layer 4      │     │    Layer 3/4     │          └──────────────────┘     └──────────────────┘     └──────────────────┘     
```
#### A. Application Load Balancer ([[ALB]]) — Layer 7

-   **Best for:** Web applications, microservices, container workloads ([[ECS]], [[EKS]]).
    
-     **Advanced Routing Capabilities:**      
        ==**Path-based==:** Route `/users` to Target Group A, `/orders` to Target Group B.        
        **Host-based:** Route `api.example.com` vs `app.example.com`.        
        **Query/Header-based:** Route traffic based on specific HTTP headers or query string parameters.
    
-  **Native Integrations:** Directly integrates with AWS WAF (Web Application Firewall), AWS Certificate Manager (ACM), and AWS Lambda as targets.
    
-  **Redirects & Fixed Responses:** Can redirect HTTP to HTTPS or serve custom error/maintenance pages directly from the ALB without hitting backends.
    

#### B. Network Load Balancer (NLB) — Layer 4

- **Best for:** Ultra-low latency requirements, TCP/UDP workloads, gaming, high-throughput financial systems handling millions of requests per second.
    
- **Key Features:**
    - Provides a **static Elastic IP (EIP)** per Availability Zone.
    - Preserves the client's source IP address natively without requiring `X-Forwarded-For` headers.
    - Handles sudden and volatile traffic spikes without pre-warming.    

#### C. Gateway Load Balancer (GWLB) — Layer 3/4

- **Best for:** Scaling and managing fleets of third-party virtual appliances (e.g., firewalls, Intrusion Detection/Prevention Systems (IDS/IPS), deep packet inspection).
    
- Uses the **GENEVE protocol** to route traffic transparently through security appliances.
    

#### D. Classic Load Balancer (CLB) — Legacy

- The original generation of AWS ELB (supports both basic Layer 4 and Layer 7).
- AWS recommends migrating existing CLBs to ALB or NLB for better feature sets, cost efficiency, and performance.
    

---

### 4. Key AWS ELB Architecture Components

1.**Listener:** Checks for connection requests on a configured port/protocol (e.g., `HTTPS:443`).
2.**Listener Rules (ALB):** Conditions (path, host, headers) and actions (forward to target group, redirect, return fixed response).    
3.**Target Group:** Logical grouping of destinations (EC2 instances, IP addresses, Lambda functions, or ECS tasks) that share health check configurations and receive forwarded traffic.

---
