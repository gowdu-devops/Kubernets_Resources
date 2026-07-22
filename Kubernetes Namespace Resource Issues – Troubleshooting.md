# <span style="color:red">Kubernetes Namespace Resource Issues – Troubleshooting Guide</span>

This guide explains the most common **Namespace Resource** issues related to **LimitRange** and **ResourceQuota**, how they occur, how to troubleshoot them, and how to resolve them in production Kubernetes clusters.

---

# <span style="color:red">Namespace Resource Validation Workflow</span>

```text
                    👨‍💻 Developer
                          │
                          ▼
               Create Deployment / Pod
                          │
                          ▼
                 Kubernetes API Server
                          │
                          ▼
                Admission Controller
                          │
         ┌────────────────┴────────────────┐
         │                                 │
         ▼                                 ▼
   Validate LimitRange             Validate ResourceQuota
         │                                 │
         ├──────────────┐                  ├──────────────┐
         │              │                  │              │
         ▼              ▼                  ▼              ▼
     Validation      Validation        Quota         Quota
      Passed          Failed          Available      Exceeded
         │              │                  │              │
         ▼              ▼                  ▼              ▼
 Resource Created   Request Rejected   Resource Created  Request Rejected
                          │
                          ▼
                    Scheduler Never Runs
```

---

# <span style="color:red">Issue 1 – Missing LimitRange</span>

## <span style="color:green">How It Occurs</span>

A developer creates a Pod without defining CPU and Memory requests or limits, and the namespace does not have a LimitRange.

### Flow Diagram

```text
Developer
     │
     ▼
Pod Created
     │
     ▼
No Requests
No Limits
     │
     ▼
BestEffort QoS
     │
     ▼
High Resource Consumption
     │
     ▼
Possible Pod Eviction
```

### Symptoms

- BestEffort QoS
- Unpredictable resource usage
- Possible eviction under node pressure

### Troubleshooting

```bash
kubectl describe pod <pod-name>
```

Look for:

```text
QoS Class: BestEffort
```

### Resolution

- Create a LimitRange.
- Configure default CPU and Memory requests/limits.
- Always define requests and limits in workloads.

---

# <span style="color:red">Issue 2 – ResourceQuota Exceeded</span>

## <span style="color:green">How It Occurs</span>

The namespace has already consumed all allocated CPU, Memory, or Pod quota.

### Flow Diagram

```text
Developer
      │
      ▼
Create Deployment
      │
      ▼
Admission Controller
      │
      ▼
Check ResourceQuota
      │
      ▼
Quota Full
      │
      ▼
Deployment Rejected
```

### Symptoms

- Deployment fails.
- Pods are not created.

### Error

```text
Error from server (Forbidden):

pods "nginx" is forbidden:

exceeded quota
```

### Troubleshooting

```bash
kubectl get resourcequota
```

```bash
kubectl describe resourcequota
```

### Resolution

- Increase namespace quota.
- Delete unused resources.
- Scale down workloads.
- Create another namespace if required.

---

# <span style="color:red">Issue 3 – Minimum CPU Validation Failed</span>

## <span style="color:green">How It Occurs</span>

The Pod requests less CPU than the minimum configured in the LimitRange.

### Flow Diagram

```text
Pod YAML
CPU Request = 50m
      │
      ▼
LimitRange
Minimum CPU = 100m
      │
      ▼
Validation Failed
      │
      ▼
Pod Rejected
```

### Error

```text
minimum cpu usage per Container is 100m
```

### Troubleshooting

```bash
kubectl describe limitrange
```

### Resolution

Update the Pod request:

```yaml
requests:
  cpu: "100m"
```

---

# <span style="color:red">Issue 4 – Maximum Memory Validation Failed</span>

## <span style="color:green">How It Occurs</span>

The Pod requests more memory than the maximum allowed by the LimitRange.

### Flow Diagram

```text
Pod Memory = 4Gi
       │
       ▼
LimitRange
Max Memory = 2Gi
       │
       ▼
Validation Failed
       │
       ▼
Pod Rejected
```

### Error

```text
maximum memory usage per Container is 2Gi
```

### Troubleshooting

```bash
kubectl describe limitrange
```

### Resolution

- Reduce the Pod's memory request/limit.
- Increase the namespace LimitRange if appropriate.

---

# <span style="color:red">Issue 5 – Namespace Resource Exhaustion</span>

## <span style="color:green">How It Occurs</span>

The namespace has consumed all available CPU, Memory, Storage, or object quotas.

### Flow Diagram

```text
Namespace
      │
      ▼
CPU Used = 8/8
Memory Used = 16Gi/16Gi
Pods = 20/20
      │
      ▼
New Deployment
      │
      ▼
Rejected
```

### Symptoms

- New Pods cannot be created.
- PVC creation fails.
- Services cannot be created.

### Troubleshooting

```bash
kubectl describe namespace <namespace>
```

```bash
kubectl describe resourcequota
```

### Resolution

- Increase the ResourceQuota.
- Delete unused Pods, PVCs, or Services.
- Move workloads to another namespace if necessary.

---

# <span style="color:red">Issue 6 – Admission Controller Rejection</span>

## <span style="color:green">How It Occurs</span>

The Admission Controller validates LimitRange and ResourceQuota policies before the resource is created. If any policy is violated, the request is rejected.

### Flow Diagram

```text
Developer
     │
     ▼
API Server
     │
     ▼
Admission Controller
     │
     ▼
Policy Validation
     │
     ├── Passed
     │      │
     │      ▼
     │ Resource Created
     │
     └── Failed
            │
            ▼
    Resource Rejected
```

### Symptoms

- Deployment or Pod is never created.
- Scheduler is never involved.

### Resolution

- Correct the LimitRange or ResourceQuota configuration.
- Update the workload's resource requests and limits to comply with namespace policies.

---

# <span style="color:red">Useful Troubleshooting Commands</span>

```bash
kubectl get ns
```

```bash
kubectl get limitrange -A
```

```bash
kubectl describe limitrange -n <namespace>
```

```bash
kubectl get resourcequota -A
```

```bash
kubectl describe resourcequota -n <namespace>
```

```bash
kubectl describe namespace <namespace>
```

```bash
kubectl get events -n <namespace> --sort-by=.metadata.creationTimestamp
```

---

# <span style="color:red">Best Practices</span>

- Create a separate namespace for each team or application.
- Apply a **LimitRange** in every namespace.
- Apply a **ResourceQuota** in every namespace.
- Always define CPU and Memory requests and limits.
- Monitor namespace resource usage regularly.
- Review quotas based on application growth.
- Avoid BestEffort Pods in production.

---

# <span style="color:red">Quick Summary</span>

| Issue | Cause | Result | Resolution |
|--------|-------|--------|------------|
| Missing LimitRange | No default requests/limits | BestEffort Pods | Configure LimitRange |
| ResourceQuota Exceeded | Namespace quota exhausted | Resource rejected | Increase quota or free resources |
| Minimum CPU Validation Failed | CPU below minimum | Pod rejected | Increase CPU request |
| Maximum Memory Validation Failed | Memory above maximum | Pod rejected | Reduce memory or update LimitRange |
| Namespace Resource Exhaustion | CPU/Memory/Storage fully used | New resources cannot be created | Increase quota or clean up resources |
| Admission Controller Rejection | Namespace policy validation failed | API request rejected | Correct LimitRange/ResourceQuota or workload configuration |
