# Advanced Container Design & Debugging Guide

This document lists container design, implementation, and production troubleshooting questions for SME/Lead/Principal-level engineers.

---

## Q1. How do you decide between single-container vs. multi-container (sidecar/ambassador/adapter)?
- Stateless services → single container per pod.  
- Cross-cutting concerns (logging, monitoring, TLS) → sidecar.  
- Legacy protocol/format conversion → adapter.  
- Dynamic external endpoints → ambassador.  
- Hybrid approach often works best.  

---

## Q2. When does the sidecar pattern become an anti-pattern?
- Multiple sidecars per service → resource bloat.  
- Increased startup time and management complexity.  
- Service mesh can centralize cross-cutting concerns at cluster level.  

---

## Q3. How do you handle stateful workloads in containers where pods are ephemeral?
- Use **StatefulSets** for stable identity.  
- Attach **PVCs** with durable storage (EBS/EFS, Ceph).  
- Apply **PodDisruptionBudgets** to maintain quorum.  
- Leverage **operators** for lifecycle automation.  
- Ensure backups & DR with snapshots or streaming.  

---

## Q4. How would you migrate stateful workloads across clusters with zero downtime?
- Use cross-region replication (Kafka MirrorMaker, DB replicas).  
- Sync replicas before cutover.  
- Cut over via DNS or service mesh once replication is healthy.  
- Validate with shadow traffic before full switch.  

---

## Q5. How would you implement blue-green or canary deployments for containerized workloads?
- **Blue-Green:** two environments; switch traffic at Ingress/load balancer.  
- **Canary:** progressive rollout (5% → 20% → 50%) using Argo Rollouts/Flagger.  
- Monitor via Prometheus/Grafana, auto-rollback on errors.  

---

## Q6. How do you handle database schema changes during blue-green or canary releases?
- Use backward-compatible migrations (expand → migrate → contract).  
- Apply feature flags for phased rollout.  
- Keep old and new schema versions running until migration completes.  
- Automate migration in CI/CD pipelines.  

---

## Q7. How do you secure sensitive data in containers?
- Store secrets in **Kubernetes Secrets** or external vaults.  
- Mount secrets as **tmpfs volumes** (not disk).  
- Use **RBAC + network policies**.  
- Automate secret rotation via CI/CD pipelines.  
- Avoid hardcoding secrets in images/env variables.  

---

## Q8. How would you design secret rotation without downtime?
- Mount versioned secrets; apps reload on change.  
- Rolling updates so pods gradually consume new secrets.  
- Ensure both old and new secrets remain valid during transition.  
- Integrate with Vault/KMS auto-rotation hooks.  

---

## Q9. How would you design observability for multi-container pods?
- Logs aggregated via sidecar (Fluentd/Vector).  
- Distributed tracing (OpenTelemetry).  
- Metrics via Prometheus exporters.  
- Correlate via labels/pod annotations.  
- Central dashboards (Grafana).  

---

## Q10. What are the different container design patterns?
- **Sidecar**, **Ambassador**, **Adapter**, **Init Container**, **Leader Election**, **Work Queue**, **Scatter-Gather**, **Sharding**, **Self-Contained**, **Daemon/Sidekick**.  
- Choice depends on workload, coupling, reusability, and operational trade-offs.  

---

## Q11. How would you design a multi-cloud container platform strategy?
- Kubernetes as common abstraction.  
- GitOps (ArgoCD/Flux) for consistent deployments.  
- Service mesh for cross-cluster routing/failover.  
- Decouple state → managed DB/storage with replication.  
- Centralized identity & policy enforcement.  

---

## Q12. How would you migrate a large monolith application into containers?
- Start with strangling monolith → extract bounded contexts.  
- Containerize monolith first for deployment consistency.  
- Gradually break critical modules into microservices.  
- Use API gateways for old/new coexistence.  
- Maintain CI/CD pipelines during migration.  

---

## Q13. How would you handle the "noisy neighbor" problem in a multi-tenant container platform?
- Resource requests/limits at pod level.  
- Pod Priority & Preemption.  
- Namespace-level resource quotas.  
- Dedicated node pools for heavy workloads.  

---

## Q14. How would you design for compliance and governance in enterprise-scale container platforms?
- Enforce policies with OPA/Gatekeeper/Kyverno.  
- Standardize base images with signed attestations.  
- Internal registry with vulnerability scanning.  
- Audit logging of K8s API requests.  
- Automate compliance checks in CI/CD.  

---

## Q15. A container is restarting in CrashLoopBackOff. How do you debug?
- `kubectl logs <pod> --previous`  
- `kubectl describe pod <pod>`  
- `kubectl exec -it <pod> -- sh`  
- Inspect runtime logs (`docker inspect`, `crictl logs`)  
- Check entrypoint/command mismatch, missing env vars, permissions  

---

## Q16. Pods are getting OOMKilled during peak traffic. How do you resolve?
- `kubectl top pod` / `kubectl describe pod`  
- Check memory requests/limits  
- Tune JVM/Node/Python memory flags  
- Use Vertical Pod Autoscaler (VPA)  

---

## Q17. Container cannot pull image in production. How do you troubleshoot?
- `kubectl describe pod` → check imagePullSecret  
- Verify image/tag exists  
- Check network/firewall and registry credentials  
- Inspect DockerHub rate limits  

---

## Q18. Service experiences intermittent latency spikes. How do you isolate root cause?
- Collect pod metrics (CPU throttling, memory pressure)  
- Node-level metrics (disk I/O, network)  
- Distributed tracing (Jaeger/Zipkin)  
- Inspect sidecar/proxy logs if service mesh enabled  
- Tcpdump/strace if needed inside container  

---

## Q19. Multi-container pod: one container ready, another failing. How to design readiness/liveness probes?
- Container-specific readiness checks  
- Init containers to delay startup until dependencies ready  
- Avoid chaining readiness of unrelated containers  
- Startup ordering via initContainers or readiness gating for sidecars  

---

## Q20. Cluster facing noisy neighbor issues: how do you debug and fix?
- Metrics for CPU throttling & evictions  
- Namespace-level quotas & LimitRanges  
- Pod Priority Classes for SLA workloads  
- Dedicated node pools for heavy workloads  

---

## Q21. Kafka/Zookeeper containers unstable in K8s. Production fixes?
- Run brokers as StatefulSets  
- Durable PVCs instead of emptyDir  
- PodAntiAffinity to spread brokers  
- Increase terminationGracePeriodSeconds  
- Debug DNS/network policies  

---

## Q22. Node outage causes pods not to reschedule. How to debug?
- Check PodDisruptionBudgets  
- PVC binding issues (zone-specific volumes)  
- Scheduler logs → taints/tolerations mismatch  
- Cluster Autoscaler scaling events  
- Manual cordon/drain to force rescheduling  

---

## Q23. Debugging CrashLoopBackOff when logs show nothing
- `kubectl describe pod <pod>` → exit codes  
- Inspect runtime: `docker inspect`, `crictl ps -a`  
- Use ephemeral debug container: `kubectl debug -it`  

---

## Q24. Pod running but not receiving traffic
- `kubectl get endpoints <svc>` → check pod IP registration  
- Validate ports: `targetPort` vs `containerPort`  
- Test local access: `kubectl exec <pod> -- curl localhost:<port>`  
- Inspect kube-proxy, iptables, CNI logs  

---

## Q25. Pods stuck in Pending state
- `kubectl describe pod` → scheduler events  
- Resource constraints / node fit  
- PVC binding, StorageClass mismatch  
- Node taints / network policies  

---

## Q26. OOMKilled after increasing memory limits
- Verify exit code 137  
- Check pod/node metrics  
- Inspect container process memory  
- Tune JVM/Node flags or app memory usage  
- Check cgroup metrics  

---

## Q27. Pod stuck in ContainerCreating
- `kubectl describe pod` → events  
- Image pull errors / registry creds  
- Volume mount failure / CSI driver logs  
- Network plugin init failure / node pressure conditions  

---

## Q28. Debug CPU throttling
- `kubectl top pod` → CPU usage vs limit  
- Check cgroup metrics: `/sys/fs/cgroup/cpu/cpu.stat`  
- Perf / flamegraph to find hotspots  
- Adjust CPU requests/limits  

---

## Q29. Container DNS resolution issue
- `kubectl exec -it <pod> -- nslookup <host>`  
- Check `/etc/resolv.conf` inside pod  
- Inspect CoreDNS logs  
- Validate `dnsPolicy`  
- Node-level network/DNS debugging  

---

## Q30. High disk I/O latency inside container
- Node I/O: `iostat -x`, `iotop`  
- Check container logs volume size  
- OverlayFS / aufs layers  
- Inode exhaustion: `df -i`  
- Use memory-backed `emptyDir` for logs if needed  

---

## Q31. Container-to-container networking bottleneck
- `kubectl exec` + iperf3 between pods  
- Trace network path: `mtr` / `traceroute`  
- Inspect node netns, tcpdump  
- Check CNI plugin performance  
- Consider eBPF dataplane CNI for high throughput  

---

## Q32. Pod evictions during production outage
- `kubectl get events` → eviction reason  
- Node logs: `journalctl -u kubelet`  
- Check `/var/lib/kubelet/pods` → disk pressure  
- Node allocatable vs capacity  
- PID pressure via cgroups  

---

## Q33. Container image bloat issue slowing deployments
- `docker history <image>` → check layer sizes  
- Multi-stage builds  
- Minimal base images (alpine/distroless)  
- Remove unnecessary packages/tools  
- Registry caching / pre-pulling  

---

## Q34. How do you make container images as small as possible?
- **Minimal base images**: alpine, distroless, scratch  
- **Multi-stage builds**: separate build/runtime  
- **Remove unnecessary files**: caches, dev artifacts  
- **Minimize layers**: combine RUN commands  
- **.dockerignore**: exclude tests, logs, .git  
- **Production dependencies only**: npm/pip install --production/no-cache-dir  
- **Static binaries**: Go/Rust → no runtime needed  
- **Compress layers/buildkit**  
- **Scan & trim**: Dive, docker-slim  

---
