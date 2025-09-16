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

## Q7. When does the sidecar pattern become an anti-pattern?
- When every service adds multiple sidecars (proxy, logging, metrics), resource usage and complexity explode.  
- At scale, a **service mesh** (Istio, Linkerd) can centralize these concerns and eliminate sidecar sprawl.  
- Example: 2000 pods × 2 sidecars = 4000 extra containers → wasted CPU/memory.  

---

## Q8. How would you migrate stateful workloads across clusters with zero downtime?
- Use **cross-cluster replication** (e.g., Kafka MirrorMaker, DB replicas).  
- Keep replicas in sync until cutover.  
- Perform **DNS-based or service discovery switch** only after validation.  
- Run dual-write temporarily if migration requires verification.  

---

## Q9. How would you design a secret rotation without downtime?
- Mount secrets as versioned files or mounted volumes.  
- Applications reload secrets dynamically when the file changes.  
- Roll out pods with new secrets while old ones remain valid for the overlap period.  
- Example: DB passwords rotated with dual authentication window.  

---

## Q10. How would you migrate stateful workloads across clusters with zero downtime?
- Use cross-region replication (e.g., Kafka MirrorMaker, DB replicas).  
- Sync replicas before cutover.  
- Cut over via DNS or service mesh once replication is healthy.  
- Validate with shadow traffic before full switch.  

---

## Q11. How would you handle the "noisy neighbor" problem in a multi-tenant container platform?
- Apply **resource requests/limits** at pod level to prevent resource hogging.  
- Use **QoS classes** in Kubernetes for critical workloads.  
- Implement **Pod Priority & Preemption** for SLA workloads.  
- Use **cgroups and CPU/memory quotas** to enforce fairness.  
- Introduce **namespace-level resource quotas** for multi-tenant isolation.  

---

## Q12. A containerized service is restarting in a CrashLoopBackOff state. How do you debug?
- Check container logs (`kubectl logs <pod> --previous`).  
- If logs aren’t visible → check stdout/stderr redirection inside container.  
- Verify entrypoint/command mismatch in Dockerfile vs. manifest.  
- Debug config/env mismatch (`kubectl describe pod <pod>`).  
- Run `kubectl exec -it <pod> -- sh` (if possible) to test startup manually.  

---

## Q13. During peak traffic, some pods are getting OOMKilled. How do you identify and fix?
- Inspect events (`kubectl describe pod`) for `OOMKilled`.  
- Check `kubectl top pod` or metrics-server for memory usage trends.  
- Validate requests/limits in deployment YAML vs. actual usage.  
- Tune JVM/Node/Python memory flags inside the container to match limits.  
- Optionally use Vertical Pod Autoscaler (VPA) recommendations.  

---

## Q14. A containerized service suddenly cannot pull images in production. What steps do you take?
- Check node events (`kubectl describe node`) for image pull errors.  
- Verify registry credentials (Secret/ServiceAccount).  
- Ensure the image tag exists and is not deleted from the registry.  
- Check for DockerHub/registry rate limits.  
- Validate cluster egress (network policies, firewall).  

---

## Q15. Your service running in containers experiences intermittent latency spikes. How do you isolate the root cause?
- Collect pod-level metrics (CPU throttling, memory pressure).  
- Check node-level issues (disk I/O, network saturation).  
- Trace requests using distributed tracing (Jaeger/Zipkin).  
- Inspect sidecar/proxy logs if service mesh is in use.  
- Run tcpdump/strace inside the container if needed.  

---

## Q16. In a multi-container pod, one container is ready while another is failing. How do you design readiness/liveness probes?
- Add container-specific readiness checks (e.g., health API, port open).  
- Use init containers to delay startup until dependencies are ready.  
- Avoid chaining the readiness of unrelated containers in the same pod.  
- For critical sidecars (e.g., envoy), enforce startup ordering via `initContainers` or readiness gating.  

---

## Q17. A production cluster is facing "noisy neighbor" issues where one team’s workloads starve others. How would you resolve this?
- Check metrics for CPU throttling & eviction events.  
- Apply **namespace-level resource quotas**.  
- Enforce **pod priority classes** for SLA workloads.  
- Add **LimitRanges** so no pod requests unbounded resources.  
- Move extremely heavy workloads into dedicated node pools.  

---

## Q18. A Kafka/Zookeeper container deployment in Kubernetes is unstable. What production fixes would you apply?
- Ensure pods are run as **StatefulSets** with stable IDs.  
- Mount durable PVCs instead of emptyDir for log/data dirs.  
- Configure `podAntiAffinity` to spread brokers/zookeepers across nodes.  
- Increase `terminationGracePeriodSeconds` to allow proper leader re-election.  
- Debug network policies or DNS resolution issues between brokers.  

---

## Q19. During a node outage, some containers fail to reschedule. How do you debug?
- Check for **PodDisruptionBudgets** blocking rescheduling.  
- Verify PVC binding issues (zone-specific volumes).  
- Inspect scheduler logs for taints/tolerations mismatch.  
- Validate cluster autoscaler scaling events.  
- Manually cordon/drain bad nodes to force rescheduling.  

---

## Q20. A pod is running but not receiving traffic via Service/Ingress. How do you isolate the problem?
- Validate `kubectl get endpoints <svc>` → confirm pod IP is registered.  
- Check `kubectl describe svc` for wrong port mapping (targetPort vs. containerPort).  
- Run `kubectl exec <pod> -- curl localhost:<port>` to confirm service works locally.  
- Use `iptables-save` or `nft list ruleset` on the node to verify kube-proxy rules.  
- If CNI issue suspected → check CNI plugin logs (`/var/log/cni`, `journalctl -u kubelet`).  

---

## Q21. Some pods are in `Pending` state indefinitely. What are the root causes you would check?
- `kubectl describe pod` → check scheduler events.  
- Resource constraints: no node fits CPU/memory requests.  
- Storage issues: PVC not bound due to a zone mismatch or missing StorageClass.  
- Node taints are not tolerated.  
- Network policies blocking CNI initialization.  
- Debug with `kubectl get events --sort-by='.lastTimestamp'`.  

---

## Q22. A pod is repeatedly getting `OOMKilled` even after increasing memory limits. What advanced steps would you take?
- Inspect container exit codes (`137`) to confirm OOM.  
- Run `kubectl top pod` and `kubectl top node` → check memory pressure.  
- Check inside container (`ps aux --sort -rss`) for memory leaks.  
- Use cgroup metrics (`cat /sys/fs/cgroup/memory/memory.max_usage_in_bytes`).  
- Tune JVM/Node heap limits (`-Xmx`, `--max-old-space-size`).  
- Enable `oom_score_adj` tuning for critical containers.  

---

## Q23. How would you troubleshoot a pod stuck in `ContainerCreating` state?
- Inspect pod events: `kubectl describe pod`.  
- Common issues:
  - Image pull errors → check secret/registry creds.  
  - Volume mount failures → check CSI driver logs.  
  - Network plugin init failures → check `/var/log/cni/`.  
- Inspect node logs: `journalctl -u kubelet` and container runtime logs.  
- Use `kubectl get node <node> -o yaml` → check conditions (DiskPressure, PIDPressure).  

---

## Q24. How do you debug CPU throttling inside containers?
- Run `kubectl top pod` → look for high CPU usage vs. limit.  
- Check cgroup throttling metrics:
  - `cat /sys/fs/cgroup/cpu/cpu.stat | grep throttled`  
- Confirm if limits are too low vs. app burst needs.  
- Use perf tools (`perf top`, `flamegraph`) inside the container for hotspots.  
- For production → prefer CPU requests without strict limits (to avoid throttling).  

---

## Q25. How would you detect and fix a DNS resolution issue inside a container?
- Exec into pod: `kubectl exec -it <pod> -- nslookup google.com`.  
- Check `/etc/resolv.conf` inside pod → confirm cluster DNS IP.  
- Inspect CoreDNS logs: `kubectl logs -n kube-system <coredns-pod>`.  
- If using custom DNS policy → check pod spec `dnsPolicy` (e.g., `ClusterFirst`, `Default`).  
- For node-level issues → test `systemd-resolved` or `/etc/hosts`.  

---

## Q26. How do you debug a container with high disk I/O latency?
- Check node I/O: `iostat -x`, `iotop`.  
- Run `du -sh /var/lib/docker/containers/*` → large logs may block I/O.  
- Inspect overlay2/aufs layers for excessive copy-on-write.  
- Use `df -i` to check inode exhaustion.  
- For high log writes → use sidecar log shippers or mount `emptyDir` with `medium: Memory`.  

---

## Q27. How do you investigate a networking bottleneck in container-to-container communication?
- Use `kubectl exec` with `iperf3` between pods.  
- Trace packet path with `mtr` or `traceroute`.  
- Inspect node network namespace: `ip netns list`, `ip netns exec <id> tcpdump`.  
- Check for kube-proxy mode (iptables vs. IPVS) performance issues.  
- If the CNI plugin is a bottleneck (e.g., Flannel), consider switching to Calico/Cilium with eBPF dataplane.  

---

## Q28. During a production outage, pods are being evicted. What advanced diagnostics would you perform?
- Run `kubectl get events` → check eviction reasons (DiskPressure, NodeNotReady, MemoryPressure).  
- On node: `dmesg | grep -i evict` or `journalctl -u kubelet`.  
- Inspect `/var/lib/kubelet/pods` → orphaned volumes causing disk pressure.  
- Use `kubectl describe node` → check allocatable vs. capacity.  
- If PID pressure → run `cat /sys/fs/cgroup/pids/pids.current`.  

---

## Q29. You suspect a container image bloat issue is slowing deployments. How would you optimize it?
- Run `docker history <image>` → check large layers.  
- Use multi-stage builds to separate build vs. runtime.  
- Switch to minimal base images (distroless, Alpine).  
- Remove unnecessary package managers/tools.  
- Enable registry-side image caching & pre-pulling on nodes.  
