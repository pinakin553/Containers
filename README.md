# Container Design Patterns

This document highlights common container design patterns that are widely used in cloud-native and microservices-based architectures. These patterns help in structuring, running, and scaling containerized applications effectively.

---

## 1. **Single Container per Host/Pod**
- **Description:** Runs one container per pod (Kubernetes) or host.
- **Use Case:** Simple services like APIs, web apps, or microservices.
- **Pros:** Clear responsibility, easy scaling, minimal coupling.
- **Cons:** May require sidecars for logging, monitoring, or networking.

---

## 2. **Sidecar Pattern**
- **Description:** A helper container runs alongside the main application container.
- **Use Case:** Logging, monitoring, service mesh proxies (e.g., Envoy in Istio).
- **Example:** NGINX sidecar serving static content, Fluentd sidecar for log shipping.
- **Pros:** Separation of concerns, reusable components.
- **Cons:** Increased resource usage, complexity in lifecycle management.

---

## 3. **Ambassador Pattern**
- **Description:** A container acts as a proxy between the application and the outside world.
- **Use Case:** Connecting to external services like databases or APIs.
- **Example:** Ambassador container handling TLS termination or database proxy.
- **Pros:** Decouples app from external connection details, easy to swap endpoints.
- **Cons:** Added network hop, potential latency.

---

## 4. **Adapter Pattern**
- **Description:** A container standardizes the interface between an application and other systems.
- **Use Case:** Log format transformation, protocol translation.
- **Example:** Adapter container converting app logs into JSON for centralized logging.
- **Pros:** Standardized outputs, interoperability.
- **Cons:** Can introduce performance overhead.

---

## 5. **Init Container Pattern**
- **Description:** A container that runs before the main app container starts.
- **Use Case:** Preparing environment, database migrations, config downloads.
- **Example:** Init container fetching secrets before app starts.
- **Pros:** Ensures app starts only after dependencies are ready.
- **Cons:** Adds startup latency.

---

## 6. **Work Queue Pattern**
- **Description:** Multiple containers consume tasks from a shared queue.
- **Use Case:** Background jobs, batch processing, async workflows.
- **Example:** Worker containers pulling tasks from RabbitMQ/Kafka/SQS.
- **Pros:** Horizontal scaling, decoupling producers/consumers.
- **Cons:** Requires queue infrastructure, at-least-once/at-most-once semantics.

---

## 7. **Sidekick Pattern**
- **Description:** A helper container provides supporting functionality but is tightly coupled to the main app.
- **Use Case:** File watchers, data synchronization, log forwarding.
- **Example:** Main app + log collector running together.
- **Pros:** Keeps app lightweight, adds missing capabilities.
- **Cons:** Strong coupling, not reusable across services.

---
