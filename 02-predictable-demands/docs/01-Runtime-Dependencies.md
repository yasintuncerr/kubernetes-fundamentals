# 📌 Overview: Understanding Runtime Dependencies

For Kubernetes to effectively manage and schedule your applications, it must understand their **runtime dependencies**. If these declared needs aren't met, your application might fail to schedule or start correctly. The two most common types of runtime dependencies are:

- 🗄️ **Storage**
- 🛠️ **Configuration**

---

## 🗄️ Storage Dependencies (Volumes)

Containers have ephemeral filesystems, meaning any data written inside a container is lost when it restarts. Kubernetes uses **Volumes** to persist or share data between containers.

### 🔹 [1.1] `emptyDir`

A temporary volume that shares the Pod's lifecycle. Ideal for sharing files between containers in the same Pod.

#### 🔧 Example Summary:

- `writer-container`: Writes logs to the volume every 5s.
- `reader-container`: Reads logs every 5s.

**YAML File:** [`01-empty-dir-volume.yaml`](./../k8s-files/runtime-depends/01-empty-dir-volume.yaml)

#### ▶️ Hands-on Steps:

1. Apply manifest:
   ```bash
   kubectl apply -f 02-predictable-demands/k8s-files/runtime-depends/01-empty-dir-volume.yaml
   ```
2. View pod status:
   ```bash
   kubectl get pods
   ```
3. Inspect logs:
   ```bash
   kubectl logs empty-dir-pod -c writer-container
   kubectl logs empty-dir-pod -c reader-container
   ```
4. Restart containers (data persists):
   ```bash
   kubectl exec empty-dir-pod -c writer-container -- kill 1
   ```
5. Delete Pod (data lost):
   ```bash
   kubectl delete pod empty-dir-pod
   kubectl apply -f 02-predictable-demands/k8s-files/runtime-depends/01-empty-dir-volume.yaml
   ```

---

### 🔹 [1.2] `PersistentVolumeClaim`

A durable volume that exists beyond the lifecycle of a single Pod.

- `PersistentVolume (PV)`: Cluster resource for storage.
- `PersistentVolumeClaim (PVC)`: Request for a portion of a PV.

**YAML File:** [`02-persistent-volume-claim.yaml`](./../k8s-files/runtime-depends/02-persistent-volume-claim.yaml)

#### ▶️ Hands-on Steps:

1. Create the hostPath directory (Minikube):
   ```bash
   mkdir -p /mnt/data
   ```
2. Apply manifest:
   ```bash
   kubectl apply -f 02-predictable-demands/k8s-files/runtime-depends/02-persistent-volume-claim.yaml
   ```
3. Check status:
   ```bash
   kubectl get pv my-pv-storage
   kubectl get pvc my-pvc-claim
   ```
4. Check persistence:
   ```bash
   kubectl exec persistent-storage-pod -- cat /app-data/status.txt
   ```
5. Delete and recreate Pod (data persists):
   ```bash
   kubectl delete pod persistent-storage-pod
   kubectl apply -f 02-predictable-demands/k8s-files/runtime-depends/02-persistent-volume-claim.yaml
   ```
6. Cleanup:
   ```bash
   kubectl delete -f 02-predictable-demands/k8s-files/runtime-depends/02-persistent-volume-claim.yaml
   ```

---

## 🛠️ Configuration Dependencies (ConfigMaps & Secrets)

Applications often need external configuration (e.g., endpoints, feature flags). Kubernetes provides:

- `ConfigMap`: Stores non-sensitive config data.
- `Secret`: Stores sensitive data (e.g., passwords).

---

### 🔹 [2.1] ConfigMap as Environment Variables

Inject values into containers as environment variables.

**YAML File:** [`03-configmap-env-vars.yaml`](./../k8s-files/runtime-depends/03-configmap-env-vars.yaml)

#### ▶️ Hands-on Steps:

1. Apply manifest:
   ```bash
   kubectl apply -f 02-predictable-demands/k8s-files/runtime-depends/03-configmap-env-vars.yaml
   ```
2. Check logs:
   ```bash
   kubectl logs configmap-env-pod
   ```
3. Cleanup:
   ```bash
   kubectl delete -f 02-predictable-demands/k8s-files/runtime-depends/03-configmap-env-vars.yaml
   ```

---

### 🔹 [2.2] ConfigMap as Volume

Mount ConfigMap data as files inside the container.

**YAML File:** [`04-configmap-volume.yaml`](./../k8s-files/runtime-depends/04-configmap-volume.yaml)

#### ▶️ Hands-on Steps:

1. Apply manifest:
   ```bash
   kubectl apply -f 02-predictable-demands/k8s-files/runtime-depends/04-configmap-volume.yaml
   ```
2. Check logs:
   ```bash
   kubectl logs configmap-volume-pod
   ```
3. Cleanup:
   ```bash
   kubectl delete -f 02-predictable-demands/k8s-files/runtime-depends/04-configmap-volume.yaml
   ```

---

### 🔹 [2.3] Secrets as Environment Variables

Store and inject sensitive values securely.

**YAML File:** [`05-secret-env-vars.yaml`](./../k8s-files/runtime-depends/05-secret-env-vars.yaml)

#### ▶️ Hands-on Steps:

1. Encode secret data:
   ```bash
   echo -n 'admin' | base64  # YWRtaW4=
   echo -n '1234' | base64   # MTIzNA==
   ```
2. Insert encoded data in YAML.
3. Apply manifest:
   ```bash
   kubectl apply -f 02-predictable-demands/k8s-files/runtime-depends/05-secret-env-vars.yaml
   ```
4. Check logs:
   ```bash
   kubectl logs secret-env-pod
   ```
5. Cleanup:
   ```bash
   kubectl delete -f 02-predictable-demands/k8s-files/runtime-depends/05-secret-env-vars.yaml
   ```
