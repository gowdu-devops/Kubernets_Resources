# <span style="color:red">Kubernetes Pod Resource Issues – Troubleshooting Guide</span>

This guide explains the most common **Pod resource-related issues**, how they occur, their symptoms, troubleshooting steps, and best practices in production environments.

---

# <span style="color:red">Pod Resource Lifecycle</span>

```text
                    Developer
                        │
                        ▼
           Deployment / Pod YAML
                        │
                        ▼
              Requests & Limits
                        │
                        ▼
           Kubernetes Scheduler
                        │
          ┌─────────────┴─────────────┐
          │                           │
          ▼                           ▼
 Enough Resources?                No Resources
          │                           │
          ▼                           ▼
    Pod Scheduled               Pod Pending
          │
          ▼
     Container Starts
          │
          ▼
   Kubelet Monitors Resources
          │
    ┌─────┴───────────────┐
    │                     │
    ▼                     ▼
CPU > Limit?        Memory > Limit?
    │                     │
    ▼                     ▼
CPU Throttled       OOMKilled
    │                     │
    ▼                     ▼
Slow Response      Container Restart
```

---

# <span style="color:red">1. Pod Pending (Insufficient Resources)</span>

## <span style="color:green">How It Occurs</span>

The Scheduler checks the Pod's **resource requests** (CPU and Memory). If no node has enough available resources, the Pod cannot be scheduled.

### Flow Diagram

```text
Developer Creates Pod
        │
        ▼
Scheduler Checks Requests
        │
        ▼
Node-1 ❌ CPU Full
Node-2 ❌ Memory Full
Node-3 ❌ CPU Full
        │
        ▼
No Suitable Node
        │
        ▼
Pod Status = Pending
```

### Symptoms

- Pod remains in **Pending** state.
- Application is not deployed.
- Scheduler reports insufficient resources.

### Troubleshooting

Check Pod status:

```bash
kubectl get pods
```

Describe the Pod:

```bash
kubectl describe pod <pod-name>
```

Typical Event:

```text
0/3 nodes are available:
Insufficient cpu
Insufficient memory
```

Check node usage:

```bash
kubectl top node
```

### Resolution

- Reduce resource requests.
- Add more worker nodes.
- Increase node size.
- Enable Cluster Autoscaler.

---

# <span style="color:red">2. OOMKilled (Memory Limit Exceeded)</span>

## <span style="color:green">How It Occurs</span>

A container uses more memory than its configured **memory limit**. The Linux kernel's Out-Of-Memory (OOM) killer terminates the container.

### Flow Diagram

```text
Container Starts
       │
       ▼
Memory Usage Increases
       │
       ▼
Limit = 512Mi
Usage = 700Mi
       │
       ▼
Memory Limit Exceeded
       │
       ▼
OOM Killer Terminates Container
       │
       ▼
Pod Restart
```

### Symptoms

- Pod restarts repeatedly.
- `STATUS` may show `CrashLoopBackOff`.
- `Last State` indicates `OOMKilled`.

### Troubleshooting

```bash
kubectl describe pod <pod-name>
```

Check:

```text
Last State:
Reason: OOMKilled
Exit Code: 137
```

View logs:

```bash
kubectl logs <pod-name> --previous
```

Monitor usage:

```bash
kubectl top pod
```

### Resolution

- Increase memory limit.
- Fix memory leaks.
- Optimize application memory usage.
- Right-size requests and limits.

---

# <span style="color:red">3. CPU Throttling</span>

## <span style="color:green">How It Occurs</span>

A container uses more CPU than its configured **CPU limit**. Instead of killing the container, Kubernetes throttles CPU usage.

### Flow Diagram

```text
Container Running
        │
        ▼
CPU Usage Increases
        │
        ▼
Limit = 500m
Usage = 900m
        │
        ▼
CPU Limit Exceeded
        │
        ▼
CPU Throttling
        │
        ▼
Application Slows Down
```

### Symptoms

- Slow API responses.
- Increased latency.
- High response times.
- Pod remains in **Running** state.

### Troubleshooting

```bash
kubectl top pod
```

Check metrics:

```bash
kubectl describe pod <pod-name>
```

Use monitoring tools:

- Prometheus
- Grafana
- Metrics Server

### Resolution

- Increase CPU limit.
- Optimize application code.
- Scale horizontally using HPA.
- Adjust requests and limits.

---

# <span style="color:red">4. Node Resource Pressure</span>

## <span style="color:green">How It Occurs</span>

The worker node runs out of CPU, Memory, or Disk resources due to high utilization.

### Flow Diagram

```text
Many Pods Running
        │
        ▼
Node Resources Exhausted
        │
        ▼
MemoryPressure
DiskPressure
PIDPressure
        │
        ▼
Kubelet Evicts Pods
```

### Symptoms

- Pods become **Evicted**.
- Node status changes to **NotReady**.
- Random application failures.

### Troubleshooting

```bash
kubectl describe node <node-name>
```

Check:

```text
Conditions:

MemoryPressure=True

DiskPressure=True
```

View node usage:

```bash
kubectl top node
```

### Resolution

- Add more worker nodes.
- Clean up disk space.
- Increase node capacity.
- Set proper resource requests.

---

# <span style="color:red">5. BestEffort Pod Eviction</span>

## <span style="color:green">How It Occurs</span>

Pods without resource requests or limits belong to the **BestEffort** QoS class. These pods are the first to be evicted when a node experiences resource pressure.

### Flow Diagram

```text
No Requests
No Limits
      │
      ▼
QoS = BestEffort
      │
      ▼
Node Under Pressure
      │
      ▼
BestEffort Pod Evicted
```

### Symptoms

- Unexpected Pod eviction.
- Frequent restarts during high node utilization.

### Troubleshooting

Check QoS:

```bash
kubectl describe pod <pod-name>
```

Look for:

```text
QoS Class: BestEffort
```

### Resolution

- Always define resource requests.
- Always define resource limits.
- Avoid BestEffort Pods in production.

---

# <span style="color:red">Useful Troubleshooting Commands</span>

```bash
kubectl get pods
```

```bash
kubectl describe pod <pod-name>
```

```bash
kubectl logs <pod-name>
```

```bash
kubectl logs <pod-name> --previous
```

```bash
kubectl top pod
```

```bash
kubectl top node
```

```bash
kubectl describe node <node-name>
```

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

---

# <span style="color:red">Best Practices</span>

- Always define CPU and Memory **requests**.
- Always define CPU and Memory **limits**.
- Monitor usage with Prometheus and Grafana.
- Configure Horizontal Pod Autoscaler (HPA).
- Use Cluster Autoscaler for dynamic node scaling.
- Avoid BestEffort Pods in production.
- Regularly review and tune resource allocations based on application metrics.

---

# <span style="color:red">Quick Summary</span>

| Issue | Cause | Pod Status | Solution |
|--------|--------|------------|----------|
| Pod Pending | Insufficient requested resources | Pending | Increase cluster capacity or reduce requests |
| OOMKilled | Memory limit exceeded | Restart / CrashLoopBackOff | Increase memory or optimize the application |
| CPU Throttling | CPU limit exceeded | Running (slow) | Increase CPU limit or optimize code |
| Node Pressure | Node resource exhaustion | Evicted | Add capacity and manage node resources |
| BestEffort Eviction | No requests or limits configured | Evicted | Configure requests and limits |
