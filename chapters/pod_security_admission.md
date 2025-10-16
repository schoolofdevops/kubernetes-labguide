# Pod Security Admission (PSA) in Kubernetes

## Lab Objective

By the end of this lab, you'll:

* Understand what PSA is and how it differs from PSP.
* Learn how to configure PSA in your cluster.
* Apply and test different security levels (`privileged`, `baseline`, `restricted`) across namespaces.
* Validate pod behavior based on enforced policies.

---

## Prerequisites

* Kubernetes v1.23+ (PSA became stable in v1.25).
* `kubectl` configured with admin access.
* A cluster (KIND or Minikube is fine for labs).
* YAML editing tool (or any editor).

---

## Step 1: Understand Pod Security Levels

Kubernetes provides **3 built-in policy levels** under PSA:

| Level        | Description                                                                            |
| ------------ | -------------------------------------------------------------------------------------- |
| `privileged` | Unrestricted pods. Allows all capabilities, similar to running as root.                |
| `baseline`   | Minimally restrictive for common container use cases. Disallows privileged containers. |
| `restricted` | Highly secure. Enforces strict security, suitable for production workloads.            |

---


## Step 2: Label Namespaces with PSA Modes

Namespaces can be configured with:

* `enforce`: blocks pods that violate the policy.
* `audit`: logs violations.
* `warn`: sends warnings but allows the pod.

### Create Namespaces

```bash
kubectl create ns secure-ns
kubectl create ns test-ns
```

### Apply PSA Labels

```bash
# Enforce restricted policy
kubectl label ns secure-ns \
  pod-security.kubernetes.io/enforce=restricted \
  pod-security.kubernetes.io/enforce-version=latest

# Warn for baseline policy
kubectl label ns test-ns \
  pod-security.kubernetes.io/warn=baseline \
  pod-security.kubernetes.io/warn-version=latest
```

---

## Step 3: Test Pod Deployment in Labeled Namespaces

### Create a Non-Compliant Pod YAML (violates restricted policy)

```yaml
# insecure-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: insecure-pod
spec:
  containers:
  - name: app
    image: nginx
    securityContext:
      allowPrivilegeEscalation: true
      capabilities:
        add: ["NET_ADMIN"]
```

### Try Applying in Different Namespaces

```bash
kubectl apply -f insecure-pod.yaml -n test-ns  # Should get warnings
kubectl apply -f insecure-pod.yaml -n secure-ns  # Should be blocked
```

> ✅ `secure-ns` will reject the pod due to `restricted` enforcement.
> ⚠️ `test-ns` will accept it but emit warnings (check via `kubectl events`).

---

## Step 4: Create a Compliant Pod

```yaml
# secure-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  containers:
  - name: app
    image: nginx
    securityContext:
      runAsNonRoot: true
      allowPrivilegeEscalation: false
```

```bash
kubectl apply -f secure-pod.yaml -n secure-ns  
```


---

## Step 5: Fixing the Compliant Pod for restricted:latest

Here’s what we must fix:

Capabilities must explicitly drop ALL.

Seccomp profile must be explicitly set to RuntimeDefault.

Run as non-root should be specified.



```yaml
# secure-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: app
    image: nginx
    securityContext:
      runAsNonRoot: true
      allowPrivilegeEscalation: false
      capabilities:
        drop: ["ALL"]
```


```bash
kubectl apply -f secure-pod.yaml -n secure-ns  # ✅ Should succeed
```

Why These Fields Matter in restricted

| Field                             | Why It's Required                                                           |
| --------------------------------- | --------------------------------------------------------------------------- |
| `runAsNonRoot`                    | Ensures containers don’t run as UID 0                                       |
| `allowPrivilegeEscalation: false` | Blocks escalated privileges even if permitted by image                      |
| `capabilities.drop: ["ALL"]`      | Drops all Linux capabilities by default                                     |
| `seccompProfile: RuntimeDefault`  | Applies the runtime’s default syscall filter to limit kernel attack surface |



---

## Step 6: View PSA Labels and Behavior

Check labels:

```bash
kubectl get ns --show-labels
```

Check pod events:

```bash
kubectl get events -n test-ns
kubectl get events -n secure-ns
```

---

## Cleanup

```bash
kubectl delete ns secure-ns test-ns
```

---

## Summary

| Feature       | PSA                 | PSP (Deprecated)      |
| ------------- | ------------------- | --------------------- |
| Type          | Built-in Admission  | API Resource          |
| Version       | Stable in v1.25     | Removed in v1.25      |
| Configuration | Namespace labels    | Cluster-wide policies |
| Use Case      | Gradual enforcement | Static blocking       |

---
