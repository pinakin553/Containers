# Container Design Patterns

This document highlights common container design patterns that are widely used in cloud-native and microservices-based architectures. These patterns help in structuring, running, and scaling containerized applications effectively.

## Q1. How do you decide between single-container vs. multi-container (sidecar/ambassador/adapter)?
- Stateless services → single container per pod.  
- Cross-cutting concerns (logging, monitoring, TLS) → sidecar.  
- Legacy protocol/format conversion → adapter.  
- Dynamic external endpoints → ambassador.  
- A hybrid approach often works best.  

---

## Q2. How do you handle stateful workloads in containers where pods are ephemeral?
- Use **StatefulSets** for stable identity.  
- Attach **PVCs** with durable storage (EBS/EFS, Ceph).  
- Apply **PodDisruptionBudgets** to maintain quorum.  
- Leverage **operators** for lifecycle automation.  
- Ensure backups & DR with snapshots or streaming.  

---

## Q3. How would you implement blue-green or canary deployments for containerized workloads?
- **Blue-Green:** two environments; switch traffic at Ingress/load balancer.  
- **Canary:** progressive rollout (5% → 20% → 50%) using Argo Rollouts/Flagger.  
- Monitor via Prometheus/Grafana, auto-rollback on errors.  
- Blue-Green → safer rollback; Canary → better progressive validation.  

---

## Q4. How do you secure sensitive data in containers?
- Store secrets in **Kubernetes Secrets** or **external vaults**.  
- Mount secrets as **tmpfs volumes** (not disk).  
- Use **RBAC + network policies** for access control.  
- Automate **secret rotation** via CI/CD pipelines.  
- Avoid hardcoding secrets in images/env variables.  

---

## Q5. How would you design observability for multi-container pods?
- Logs aggregated via sidecar (Fluentd/Vector).  
- Distributed tracing with **OpenTelemetry** across containers.  
- Metrics exposed by sidecar exporters → Prometheus.  
- Correlate using **trace IDs** injected into logs/metrics.  
- Ensures full visibility across app + helper containers.  

---

## Q6. What are the different container design patterns and when do you use them?
- **Single Container per Pod:** Best for stateless services (simple APIs).  
- **Sidecar:** Adds logging, monitoring, or proxy (e.g., Envoy, Fluentd).  
- **Ambassador:** Handles outbound connections to external DBs/APIs.  
- **Adapter:** Normalizes logs/protocols for standard outputs.  
- **Init Container:** Prepares environment (fetch configs, migrations).  
- **Work Queue / Worker:** Multiple workers pull jobs from a queue.  
- **Sidekick:** Tightly coupled helper container (e.g., file sync, log forwarder).  
- **Daemonset (Kubernetes):** Runs infra agents (Prometheus Node Exporter, Fluent Bit) on every node.  
- **Scatter-Gather:** Fan-out requests to multiple workers and aggregate responses.  
- **Data Loader Sidecar:** Preloads data/models/configs into shared volumes.  
- Pattern choice depends on workload requirements, operational complexity, and trade-offs (performance, coupling, reusability).  

---
