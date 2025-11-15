servicemesh is different from kubernetes service. <br>
They solve different problems even though both deal with networking. <br>

Kubernetes service: <br>
- Exposes stable virtual IPs for accessible stable virtual pods.
- Basic load balancing and service discovery.
- Handles basic load balancing and routing. 

### Service mesh: <br>

- Handles **secure, reliable, and observable communication** between services. <br>
- Adds advanced logic like: <br>
    mTLS (encryption) <br>
    Retries, timeouts, circuit breaking <br>
    Traffic shifting (canary, A/B tests) <br>
    Tracing, metrics <br>
    Policy enforcement <br>

**Higher-level networking; not part of Kubernetes core.** <br>

| Feature        | Kubernetes Service | Service Mesh                      |
| -------------- | ------------------ | --------------------------------- |
| Network Layer  | L3/L4              | L7 (HTTP/gRPC aware)              |
| Discovery      | Basic              | Advanced                          |
| Load Balancing | Round-robin        | Weighted, locality-aware, retries |
| Security       | None               | mTLS, identities, policies        |
| Observability  | Minimal            | Full metrics + tracing            |

**ServiceMesh does not replace kubernete services.** <br>
It runs on the top of that. <br>

**It works as a sidecar proxies.** <br>
example: Istio, Linkerd (older versions), Consul, Kuma. <br> <br>

Istio:
-----
its a traffic manager and security manager for your microservice. <br>

- Traffic control
- Security
- Observability

Istio + Prometheus integration:
------------------------------

Prometheus only can collect Node Metrics, Pod metrics. <br>
Istio can generate detailed telemetry which monitoring tools can use. <br>

## Why Istio needs Prometheus? <br>

Istio sidecar proxies (Envoy) generate rich metrics for: <br>
### ✔ Traffic (Canary,Blue/Green,A/B testing,Fault injection,Circuit breaking)
- request count
- request duration
- success/error rate
- per-path metrics
- per-service metrics
### ✔ Reliability
- retries
- timeouts
- circuit breaker triggers
- connection errors
### ✔ Security
- mTLS handshake
- certificate failures
- unauthorized requests
### ✔ Mesh internals
- sidecar uptime
- pilot synchronization
- cluster-wide mesh health
- Prometheus scrapes all this data


