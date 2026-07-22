# Kubernetes Namespace Resources Workflow

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
          ┌────────────────────┴────────────────────┐
          │                                         │
          ▼                                         ▼
   Check LimitRange                         Check ResourceQuota
          │                                         │
          │                                         │
   Requests/Limits Valid?                 Namespace Quota Available?
          │                                         │
     ┌────┴────┐                             ┌──────┴──────┐
     │         │                             │             │
    Yes       No                            Yes           No
     │         │                             │             │
     │         ▼                             │             ▼
     │  Resource Rejected                    │     Quota Exceeded
     │  Validation Error                     │     Resource Rejected
     │                                       │
     └───────────────┬───────────────────────┘
                     │
                     ▼
             Resource Successfully Created
                     │
                     ▼
          Scheduler Reads Resource Requests
                     │
                     ▼
           Suitable Worker Node Found?
              │                    │
             Yes                  No
              │                    │
              ▼                    ▼
        Pod Scheduled        Pod Pending
              │
              ▼
        Kubelet Starts Pod
              │
              ▼
      Application Running Successfully
```

---

# Workflow Explanation

### Step 1 – Developer
Creates a Pod or Deployment manifest.

↓

### Step 2 – API Server
Receives the request.

↓

### Step 3 – Admission Controller
Before creating the resource, Kubernetes validates namespace policies.

It checks:

- ✅ LimitRange
- ✅ ResourceQuota

↓

### Step 4 – LimitRange Validation

Checks:

- Minimum CPU
- Maximum CPU
- Minimum Memory
- Maximum Memory
- Default Requests
- Default Limits

If validation fails:

```text
minimum cpu usage per Container is 100m
```

Resource is rejected.

↓

### Step 5 – ResourceQuota Validation

Checks:

- CPU quota
- Memory quota
- Pod count
- PVC count
- Service count
- Storage quota

If quota is exceeded:

```text
pods "nginx" is forbidden:
exceeded quota
```

↓

### Step 6 – Scheduler

Only after passing both validations does the Scheduler run.

The Scheduler:

- Reads Pod Requests
- Finds a suitable Worker Node
- Schedules the Pod

↓

### Step 7 – Kubelet

The Kubelet:

- Pulls the container image
- Starts the container
- Enforces CPU and Memory limits during runtime

---

# Easy Way to Remember

```text
Developer
      │
      ▼
API Server
      │
      ▼
Admission Controller
      │
      ├── LimitRange ✔
      ├── ResourceQuota ✔
      │
      ▼
Resource Created
      │
      ▼
Scheduler
      │
      ▼
Worker Node
      │
      ▼
Kubelet
      │
      ▼
Running Pod
```

## Interview Tip

A common interview question is:

**"When are LimitRange and ResourceQuota applied?"**

**Answer:**
- **LimitRange** and **ResourceQuota** are validated by the **Admission Controller** before the resource is created.
- If validation succeeds, the API Server stores the resource and the Scheduler schedules the Pod.
- If validation fails, the request is rejected immediately and the Scheduler is never involved.
