# 📌 Overview: Understanding Resource Profiles

For Kubernetes to schedule Pods efficiently and ensure stable performance, it must know the resource needs of each container. This is done by defining **resource profiles** using `requests` and `limits`.

---

## 🧮 Resource Types

- **`requests`**: Minimum amount of resources a container needs to run. Kubernetes uses this value for scheduling.
- **`limits`**: Maximum amount of resources a container is allowed to use.

---

## ⚖️ Compressible vs Incompressible Resources

| Type           | Example | Behavior when exceeded                 |
|----------------|---------|----------------------------------------|
| Compressible   | CPU     | Container is throttled, not killed     |
| Incompressible | Memory  | Container is **OOMKilled**             |

---

# 1️⃣ Quality of Service (QoS) Classes

Kubernetes assigns a **QoS class** based on the presence and equality of resource `requests` and `limits`:

### 🔒 Guaranteed
- All containers have equal `requests` and `limits` for **both CPU and memory**.
- Highest priority, last to be killed.

### 🚀 Burstable
- At least one container has `limits` > `requests`.
- Medium priority.

### 🧪 Best-Effort
- No `requests` or `limits` specified at all.
- Lowest priority, first to be evicted.

---

## 1.1 🔒 `Guaranteed` QoS Pod

This is the most predictable and **recommended configuration for production workloads**.

### ✅ Criteria
- CPU and memory `requests` **equal** to `limits`.

### 📄 YAML File  
[01-guaranteed-pod.yaml](./../k8s-files/resource-profiles/01-guaranteed-pod.yaml)

### 🛠 Hands-on

```bash
# Create the Pod
kubectl apply -f 02-predictable-demands/k8s-files/resource-profiles/01-guaranteed-pod.yaml

# Check the QoS Class
kubectl describe pod guaranteed-pod
# Look for: QoS Class: Guaranteed

# Cleanup
kubectl delete pod guaranteed-pod
```

---

## 1.2 🚀 `Burstable` QoS Pod

Useful when your app might need **more resources occasionally**.

### ✅ Criteria
- `requests` set, but `limits` > `requests`.

### 📄 YAML File  
[02-burstable-pod.yaml](./../k8s-files/resource-profiles/02-burstable-pod.yaml)

### 🛠 Hands-on

```bash
# Create the Pod
kubectl apply -f 02-predictable-demands/k8s-files/resource-profiles/02-burstable-pod.yaml

# Check the QoS Class
kubectl describe pod burstable-pod
# Look for: QoS Class: Burstable

# Cleanup
kubectl delete pod burstable-pod
```

---

## 1.3 🧪 `Best-Effort` QoS Pod

Should be used for **non-critical or batch** jobs.

### ✅ Criteria
- No `requests` or `limits` defined.

### 📄 YAML File  
[03-best-effort-pod.yaml](./../k8s-files/resource-profiles/03-best-effort-pod.yaml)

### 🛠 Hands-on

```bash
# Create the Pod
kubectl apply -f 02-predictable-demands/k8s-files/resource-profiles/03-best-effort-pod.yaml

# Check the QoS Class
kubectl describe pod best-effort-pod
# Look for: QoS Class: Best-Effort

# Cleanup
kubectl delete pod best-effort-pod
```

---