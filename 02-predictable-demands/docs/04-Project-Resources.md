# 📌 Overview: Understanding Project Resources & Capacity Planning
While developers define resoure needs for individual Pods, administrators need tools to manage resources for an entire project or namespace. This ensures fair use of cluster resources and prevents a single workload from starving others in multi-tenant environments.

Kubernetes provides two main mechanisms:

- ``ResourceQuota``: Controls **total** usage per namespace
- ``LimitRange``: Controls **per-Pod or per-container** resource requests and limits.

## [1] ResourceQuota: Limiting Total Consumption

Contstrain the aggregate resource consumption in a namespace.

**YAML File:** ['01-resource-quota.yaml](./../k8s-files/project-resources/01-resource-quota.yaml)


#### ▶️ Hands-on Steps:

1. Apply manifest
    ```bash
    kubectl apply -f 02-predictable-demands/k8s-files/project-resources/01-resource-quota.yaml
    ```
2. Verify quota status
    ```bash
    kubectl describe quota my-quota --namespace=quota-example
    ```
3. Launch two Pods(should succeed):
    ```bash
    # Pod1 Request  500Mi Memory and 500m CPU resource
    kubectl run pod1 --image=busybox  --namespace=quota-example --restart=Never \
    --overrides='{
    "spec": {
      "containers": [{
        "name": "app",
        "image": "busybox",
        "command": ["/bin/sh", "-c", "sleep 3600"],
        "resources": {
          "requests": {
            "memory": "500Mi"
            },
          "limits": {
            "cpu": "500m"
            }
            }
        }]
        }
    }'



    # Pod2 Request  500Mi Memory and 500m CPU resource
    kubectl run pod2 --image=busybox --namespace=quota-example --restart=Never \
    --overrides='{
        "spec": {
        "containers": [{
            "name": "app",
            "image": "busybox",
            "command": ["/bin/sh", "-c", "sleep 3600"],
            "resources": {
            "requests": {
                "memory": "500Mi"           
            },
            "limits": {
                "cpu": "500m"
            }
            }
        }]
        }
    }'
    
    ```

4. Verify quotas status again
    ```bash
    kubectl describe quota my-quota --namespace=quota-example
    ```
    Now, if you check the **my-quota** namespace, you should see that the **Used** column no longer shows 0 — it reflects the resources consumed by your running pod.


4.  Try a third Pod (should fail)
    ```bash
    # Pod3 Request  2Gi Memory and 2 CPU resource. it should be exceed namaspace quotas
    kubectl run pod3 --image=busybox --namespace=quota-example --restart=Never \
    --overrides='{
        "spec": {
        "containers": [{
            "name": "app",
            "image": "busybox",
            "command": ["/bin/sh", "-c", "sleep 3600"],
            "resources": {
            "requests": {
                "memory": "2Gi"           
            },
            "limits": {
                "cpu": "2"
            }
            }
        }]
        }
    }'
    ```

    Expected error:
    ```md
    Error from server (Forbidden): pods "pod3" is forbidden: exceeded quota: my-quota, requested: limits.cpu=2,pods=1,requests.memory=2Gi, used: limits.cpu=1,pods=2,requests.memory=1000Mi, limited: limits.cpu=2,pods=2,requests.memory=1Gi
    ```

5. Cleanup
    ```bash
    # When you delete namespace all resources under that namespaces are removed
    kubectl delete namespace quota-example
    ```

## [2] LimitRange: Enforcing Defaults & Bounds

Assign default requests and limits, and enforce min/max per container

**YAML File:** ['02-limit-range.yaml](./../k8s-files/project-resources/02-limit-range.yaml)


#### ▶️ Hands-on Steps:

1. Apply manifest
    ```bash
    kubectl apply -f 02-predictable-demands/k8s-files/project-resources/02-limit-range.yaml
    ```
2. Inspect the Pod
    ```bash
    kubectl describe pod limitrange-pod --namespace=limit-example
    ```
    In the "Containers" section, you should see default limits/requests applied

5. Cleanup
    ```bash
    kubectl delete namespace limit-example
    ```