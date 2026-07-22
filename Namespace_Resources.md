# <span style="color:red">Kubernetes Namespace Resources (LimitRange & ResourceQuota)</span>

---

# <span style="color:red">What are Namespace Resources?</span>

## <span style="color:green">Definition</span>

Namespace Resources are policies that control **CPU, Memory, Storage, and the number of Kubernetes objects** within a specific namespace.

Instead of controlling a single Pod, they manage resources for **all workloads inside a namespace**.

Kubernetes provides two main namespace-level resource management objects:

- **LimitRange**
- **ResourceQuota**

---

# <span style="color:red">Why Do We Need Namespace Resources?</span>

## <span style="color:green">Problem Statement</span>

Imagine a production Kubernetes cluster shared by multiple teams:

- Team A
- Team B
- Team C

If no namespace resource policies are configured:

- Team A creates many Pods.
- Team A consumes all CPU and Memory.
- Team B cannot deploy applications.
- Team C's applications become slow.
- The cluster becomes unstable.

Namespace resources prevent one team from consuming all cluster resources.

---

# <span style="color:red">Namespace Resource Architecture</span>

```text
                 Kubernetes Cluster
                         │
      ┌──────────────────┼──────────────────┐
      │                  │                  │
      ▼                  ▼                  ▼
 Namespace-A       Namespace-B       Namespace-C
      │                  │                  │
      │                  │                  │
 LimitRange         LimitRange         LimitRange
 ResourceQuota      ResourceQuota      ResourceQuota
      │                  │                  │
      ▼                  ▼                  ▼
 Pods               Pods               Pods
```

---

# <span style="color:red">Namespace-Level Resources</span>

| Resource | Purpose |
|----------|---------|
| LimitRange | Defines default, minimum, and maximum CPU/Memory for containers |
| ResourceQuota | Limits the total resources and object count in a namespace |

---

# <span style="color:red">LimitRange</span>

## <span style="color:green">Definition</span>

A **LimitRange** sets default, minimum, and maximum resource values for containers or Pods within a namespace.

If a developer does not specify resource requests or limits, Kubernetes automatically applies the defaults defined in the LimitRange.

---

## <span style="color:green">Why Use LimitRange?</span>

- Enforces resource standards.
- Prevents containers from running without requests and limits.
- Prevents extremely high or low resource allocations.
- Ensures consistency across applications.

---

## <span style="color:green">Example</span>

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: app-limit
spec:
  limits:
  - type: Container

    default:
      cpu: "1"
      memory: 1Gi

    defaultRequest:
      cpu: "500m"
      memory: 512Mi

    min:
      cpu: "100m"
      memory: 128Mi

    max:
      cpu: "2"
      memory: 2Gi
```

---

# <span style="color:red">ResourceQuota</span>

## <span style="color:green">Definition</span>

A **ResourceQuota** limits the total amount of resources and the number of Kubernetes objects that can exist in a namespace.

---

## <span style="color:green">Why Use ResourceQuota?</span>

- Prevents one team from consuming all cluster resources.
- Controls namespace capacity.
- Improves resource sharing.
- Supports multi-tenant Kubernetes clusters.

---

## <span style="color:green">Example</span>

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: team-a-quota

spec:
  hard:

    requests.cpu: "4"

    requests.memory: 8Gi

    limits.cpu: "8"

    limits.memory: 16Gi

    pods: "20"

    services: "10"

    persistentvolumeclaims: "5"
```

---

# <span style="color:red">LimitRange vs ResourceQuota</span>

| Feature | LimitRange | ResourceQuota |
|----------|------------|---------------|
| Scope | Individual Pod/Container | Entire Namespace |
| Controls | Default, Min, Max Resources | Total Resources |
| CPU | ✅ | ✅ |
| Memory | ✅ | ✅ |
| Storage | ❌ | ✅ |
| Pod Count | ❌ | ✅ |
| Service Count | ❌ | ✅ |
| PVC Count | ❌ | ✅ |

---

# <span style="color:red">How They Work Together</span>

```text
Developer Creates Deployment
            │
            ▼
Namespace
            │
            ▼
LimitRange
(Default CPU & Memory Applied)
            │
            ▼
ResourceQuota
(Check Namespace Capacity)
            │
            ▼
Pod Created
```

---

# <span style="color:red">Real-Time Production Scenario</span>

## <span style="color:green">Scenario 1 - No ResourceQuota</span>

Namespace:

```
production
```

Team A deploys

- 100 Pods
- 100 CPU
- 200 GB Memory

Result

- Team B cannot deploy.
- Scheduler reports insufficient resources.
- Cluster performance degrades.

---

## <span style="color:green">Solution</span>

Apply a ResourceQuota.

Example

```
pods: 20

requests.cpu: 4

limits.cpu: 8
```

Now Team A cannot exceed the allocated resources.

---

# <span style="color:red">Scenario 2 - No LimitRange</span>

Developer creates:

```yaml
containers:
- image: nginx
```

No Requests.

No Limits.

Result

- Container consumes unlimited resources.
- Node performance decreases.
- Other applications become slow.

---

## <span style="color:green">Solution</span>

LimitRange automatically assigns default CPU and Memory values.

---

# <span style="color:red">Troubleshooting</span>

## <span style="color:green">Check ResourceQuota</span>

```bash
kubectl get resourcequota -n production
```

---

```bash
kubectl describe resourcequota -n production
```

---

## <span style="color:green">Check LimitRange</span>

```bash
kubectl get limitrange -n production
```

---

```bash
kubectl describe limitrange -n production
```

---

## <span style="color:green">Check Namespace Resources</span>

```bash
kubectl describe namespace production
```

---

# <span style="color:red">Common Errors</span>

### ResourceQuota Exceeded

```text
Error from server (Forbidden):

pods "nginx" is forbidden:

exceeded quota:

production-quota
```

---

### LimitRange Validation Error

```text
Error:

minimum cpu usage per Container is 100m
```

---

# <span style="color:red">Best Practices</span>

## <span style="color:green">Recommendations</span>

- Create a separate namespace for each team.
- Apply a LimitRange in every namespace.
- Apply a ResourceQuota in every namespace.
- Monitor namespace resource usage regularly.
- Prevent unlimited resource consumption.

---

# <span style="color:red">Interview Questions</span>

## <span style="color:green">Q1. What are Namespace Resources?</span>

Namespace Resources are Kubernetes policies that manage CPU, Memory, Storage, and object usage for all workloads within a namespace.

---

## <span style="color:green">Q2. What is the difference between LimitRange and ResourceQuota?</span>

**LimitRange** controls the default, minimum, and maximum CPU and Memory for individual Pods or Containers.

**ResourceQuota** controls the total resources and object counts available across the entire namespace.

---

## <span style="color:green">Q3. Can we use both LimitRange and ResourceQuota together?</span>

Yes. In production, they are commonly used together.

- **LimitRange** ensures each Pod has appropriate resource requests and limits.
- **ResourceQuota** ensures the namespace as a whole stays within its allocated resource budget.

---

# <span style="color:red">Easy Way to Remember</span>

- **Namespace** → Creates isolation between teams or applications.
- **LimitRange** → Controls resources for **each Pod/Container**.
- **ResourceQuota** → Controls resources for the **entire Namespace**.
