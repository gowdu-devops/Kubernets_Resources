# <span style="color:red">Kubernetes Pod-Level Resource Architecture</span>

```text
                         👨‍💻 Developer
                               │
                               │
                 Creates Deployment YAML
                               │
                               ▼
┌────────────────────────────────────────────────────────────┐
│                    Deployment Manifest                     │
│                                                            │
│  Requests                     Limits                       │
│  CPU : 500m                  CPU : 1 Core                  │
│  Memory : 512Mi              Memory : 1Gi                  │
└────────────────────────────────────────────────────────────┘
                               │
               Scheduler Reads ONLY Requests
                               │
                               ▼
┌────────────────────────────────────────────────────────────┐
│                 Kubernetes Scheduler                       │
│                                                            │
│ Checks Node Capacity                                       │
│ • CPU Available?                                           │
│ • Memory Available?                                        │
└────────────────────────────────────────────────────────────┘
                     │                     │
                     │                     │
                 Enough Resources      No Resources
                     │                     │
                     ▼                     ▼
            Pod Scheduled            Pod Pending
                     │
                     ▼
┌────────────────────────────────────────────────────────────┐
│                     Worker Node                            │
│                                                            │
│  ┌────────────────────────────────────────────────────┐    │
│  │                     Pod                            │    │
│  │                                                    │    │
│  │  ┌──────────────────────────────────────────────┐  │    │
│  │  │                Container                     │  │    │
│  │  │                                              │  │    │
│  │  │ Requests                                     │  │    │
│  │  │ CPU    : 500m                                │  │    │
│  │  │ Memory : 512Mi                               │  │    │
│  │  │                                              │  │    │
│  │  │ Limits                                       │  │    │
│  │  │ CPU    : 1 Core                              │  │    │
│  │  │ Memory : 1Gi                                 │  │    │
│  │  └──────────────────────────────────────────────┘  │    │
│  └────────────────────────────────────────────────────┘    │
└────────────────────────────────────────────────────────────┘
                     │
                     ▼
           Kubelet Monitors Resources
                     │
         ┌───────────┴───────────┐
         │                       │
         ▼                       ▼
 CPU > Limit?             Memory > Limit?
         │                       │
        Yes                     Yes
         │                       │
         ▼                       ▼
 CPU Throttling            OOMKilled
         │                       │
         ▼                       ▼
 Application Slow      Container Restarted
```
