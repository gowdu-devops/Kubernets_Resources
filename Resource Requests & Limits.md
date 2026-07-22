# <span style="color:red">Kubernetes Resources (CPU & Memory) – Complete Notes</span>

---

# <span style="color:red">What are Kubernetes Resources?</span>

## <span style="color:green">Definition</span>

Resources in Kubernetes define how much **CPU** and **Memory (RAM)** a container requires to run efficiently.

Kubernetes uses these resource values to:

- Schedule Pods on suitable Nodes.
- Prevent one application from consuming all cluster resources.
- Ensure fair resource sharing among applications.
- Maintain application stability and performance.

The two primary resources are:

- **CPU**
- **Memory (RAM)**

---

# <span style="color:red">Why Do We Need Resources?</span>

## <span style="color:green">Problem Without Resources</span>

If resource requests and limits are not configured:

- One Pod may consume all CPU.
- One Pod may consume all Memory.
- Other Pods may become slow or fail.
- Node performance degrades.
- Cluster becomes unstable.
- Critical applications may stop responding.

---

# <span style="color:red">Types of Resources</span>

## <span style="color:green">CPU</span>

CPU represents the processing power allocated to a container.

Example:

```yaml
cpu: "500m"
```

500m = 0.5 CPU Core

Examples:

| Value | Meaning |
|--------|----------|
| 100m | 0.1 Core |
| 250m | 0.25 Core |
| 500m | 0.5 Core |
| 1000m | 1 CPU Core |
| 2000m | 2 CPU Cores |

---

## <span style="color:green">Memory</span>

Memory defines how much RAM a container can use.

Examples

```yaml
memory: "256Mi"
```

```yaml
memory: "1Gi"
```

Common Units

| Value | Meaning |
|--------|----------|
| 128Mi | 128 MB |
| 256Mi | 256 MB |
| 512Mi | 512 MB |
| 1Gi | 1024 MB |
| 2Gi | 2048 MB |

---

# <span style="color:red">CPU vs Memory</span>

| CPU | Memory |
|------|---------|
| Processing Power | RAM |
| Can be throttled | Cannot be throttled |
| Measured in millicores | Measured in Mi/Gi |
| Scheduler checks requests | Scheduler checks requests |

---

# <span style="color:red">Resource Requests</span>

## <span style="color:green">Definition</span>

A **Request** is the minimum amount of CPU and Memory that Kubernetes guarantees for a container.

The Scheduler uses requests to decide on which node the Pod should run.

Example

```yaml
resources:
  requests:
    cpu: "500m"
    memory: "512Mi"
```

Meaning

- Minimum CPU = 500m
- Minimum Memory = 512Mi

---

# <span style="color:red">Resource Limits</span>

## <span style="color:green">Definition</span>

A **Limit** is the maximum amount of CPU and Memory a container is allowed to use.

Example

```yaml
resources:
  limits:
    cpu: "1"
    memory: "1Gi"
```

Meaning

- Maximum CPU = 1 Core
- Maximum Memory = 1Gi

---

# <span style="color:red">Requests vs Limits</span>

| Request | Limit |
|----------|-------|
| Minimum Resource | Maximum Resource |
| Used by Scheduler | Enforced by Kubernetes |
| Guarantees Resources | Prevents Overuse |
| Required for Scheduling | Controls Runtime Usage |

---

# <span style="color:red">How Kubernetes Uses Resources</span>

```text
Developer Creates Deployment
            │
            ▼
Request = 500m CPU
Limit = 1 CPU
            │
            ▼
Scheduler Checks Nodes
            │
            ▼
Node Has Enough Resources?
       │             │
      Yes            No
       │             │
       ▼             ▼
Pod Scheduled    Pod Pending
```

---

# <span style="color:red">Complete YAML Example</span>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx

spec:
  replicas: 2

  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      labels:
        app: nginx

    spec:
      containers:
      - name: nginx

        image: nginx

        resources:

          requests:
            cpu: "500m"
            memory: "512Mi"

          limits:
            cpu: "1"
            memory: "1Gi"
```

---

# <span style="color:red">Real-Time Production Scenario</span>

## <span style="color:green">Scenario</span>

A company deploys:

- Frontend
- Backend
- Database

The Backend application starts consuming excessive CPU because of an infinite loop.

Without Limits:

- Backend uses all CPU.
- Frontend becomes slow.
- Database performance decreases.
- Users experience downtime.

With Resource Limits:

- Backend is restricted to its configured CPU.
- Frontend continues working.
- Database remains stable.

---

# <span style="color:red">Common Resource Issues</span>

| Issue | Cause |
|---------|--------|
| Pod Pending | Insufficient requested CPU/Memory |
| OOMKilled | Memory limit exceeded |
| CPU Throttling | CPU limit exceeded |
| Slow Application | Low CPU request |
| Node Resource Exhaustion | No resource limits |

---

# <span style="color:red">How to Troubleshoot Resource Issues</span>

## <span style="color:green">Check Pod Status</span>

```bash
kubectl get pods
```

---

## <span style="color:green">Describe Pod</span>

```bash
kubectl describe pod <pod-name>
```

Look for:

- Insufficient CPU
- Insufficient Memory
- FailedScheduling

---

## <span style="color:green">Check Events</span>

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

Example

```text
0/3 nodes are available:
Insufficient cpu.
```

---

## <span style="color:green">View Resource Usage</span>

```bash
kubectl top pod
```

```bash
kubectl top node
```

---

# <span style="color:red">Best Practices</span>

## <span style="color:green">Recommendations</span>

✅ Always define Requests.

✅ Always define Limits.

✅ Monitor CPU and Memory usage.

✅ Use appropriate values based on application requirements.

✅ Avoid setting limits too low or too high.

---

# <span style="color:red">Interview Questions</span>

## <span style="color:green">Q1. What are Kubernetes Resources?</span>

Resources define the CPU and Memory allocated to containers.

---

## <span style="color:green">Q2. What is a Resource Request?</span>

A Request is the minimum CPU and Memory guaranteed to a container and is used by the Scheduler to place Pods on Nodes.

---

## <span style="color:green">Q3. What is a Resource Limit?</span>

A Limit is the maximum CPU and Memory a container can consume during runtime.

---

## <span style="color:green">Q4. What happens if a Pod requests more CPU than available?</span>

The Pod remains in the **Pending** state until sufficient resources become available.

---

## <span style="color:green">Q5. What happens if a container exceeds its Memory Limit?</span>

The container is terminated by Kubernetes and the Pod status typically shows **OOMKilled**.

---

## <span style="color:green">Q6. What happens if a container exceeds its CPU Limit?</span>

The CPU usage is throttled (limited). The container is not terminated, but application performance may degrade.

---

# <span style="color:red">Quick Revision</span>

| Resource | Purpose |
|-----------|----------|
| CPU | Processing Power |
| Memory | RAM |
| Request | Minimum Guaranteed Resource |
| Limit | Maximum Allowed Resource |
| Scheduler | Uses Requests |
| Runtime | Enforces Limits |
| Memory Limit Exceeded | OOMKilled |
| CPU Limit Exceeded | CPU Throttling |

---

# <span style="color:red">Easy Way to Remember</span>

- **Request** → "Please reserve this much CPU and Memory for me."
- **Limit** → "Do not allow me to use more than this amount."
- **CPU** → If exceeded, Kubernetes **throttles** the container.
- **Memory** → If exceeded, Kubernetes **kills** the container (OOMKilled).
