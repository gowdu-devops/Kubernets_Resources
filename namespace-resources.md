# Kubernetes Namespace Resources

## 1. Introduction

- What are Namespace Resources?
- Why do we need Namespace Resources?
- Pod-level vs Namespace-level resources

---

## 2. Namespace Resource Architecture

- Kubernetes Cluster
- Namespace
- LimitRange
- ResourceQuota
- Pods

(Architecture Diagram)

---

## 3. LimitRange

- What is LimitRange?
- Why do we need it?
- Default Requests
- Default Limits
- Minimum Resources
- Maximum Resources
- Container-level enforcement
- YAML Example
- Production Scenario

---

## 4. ResourceQuota

- What is ResourceQuota?
- Why do we need it?
- CPU Quota
- Memory Quota
- Pod Quota
- PVC Quota
- Service Quota
- Secret Quota
- ConfigMap Quota
- YAML Example
- Production Scenario

---

## 5. Complete Namespace Resource Flow

Developer
        │
        ▼
Namespace
        │
        ├──────── LimitRange
        │
        ├──────── ResourceQuota
        │
        ▼
Deployment
        │
        ▼
Pod Created
        │
        ▼
Scheduler
        │
        ▼
Node

---

## 6. How LimitRange Works

Flow Diagram

Developer
      │
      ▼
Pod Without Resources
      │
      ▼
Namespace Has LimitRange
      │
      ▼
Default CPU/Memory Applied
      │
      ▼
Pod Created

---

## 7. How ResourceQuota Works

Flow Diagram

Developer
      │
      ▼
Create Pod
      │
      ▼
Admission Controller
      │
      ▼
Check ResourceQuota
      │
      ├──────── Enough Quota
      │              │
      │              ▼
      │         Pod Created
      │
      └──────── Quota Exceeded
                     │
                     ▼
          Pod Rejected

---

## 8. Production Scenarios

Scenario 1

Developer forgot Requests and Limits

↓

LimitRange automatically assigns defaults.

---

Scenario 2

Team deployed 150 Pods

↓

ResourceQuota blocks further deployments.

---

Scenario 3

Namespace exceeded CPU quota

↓

Pod creation rejected.

---

## 9. Common Errors

Exceeded ResourceQuota

Exceeded CPU

Exceeded Memory

Exceeded Pod Count

Minimum CPU Validation Failed

Maximum Memory Validation Failed

---

## 10. Troubleshooting

kubectl get limitrange

kubectl describe limitrange

kubectl get resourcequota

kubectl describe resourcequota

kubectl describe namespace

kubectl get events

---

## 11. Best Practices

One Namespace per Team

LimitRange in Every Namespace

ResourceQuota in Every Namespace

Monitor Namespace Usage

Avoid Unlimited Resources

---

## 12. Interview Questions

What is LimitRange?

What is ResourceQuota?

Difference between LimitRange and ResourceQuota?

Can we use both together?

How does ResourceQuota work?

What happens if the quota is exceeded?

How do you troubleshoot ResourceQuota issues?

---

## 13. Summary
