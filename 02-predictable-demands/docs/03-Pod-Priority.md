# 📌 Understanding Pod Priority & Preemption

While Quality of Service (QoS) affects how the Kubelet evicts POds under resource pressure, **Pod Priority** impacts the **scheduler's** decisions by influencing which Pods are scheduled **first**

This system is built on two concepts:

- **PriorityClass**: Cluster-scpoed resource that maps a name to a priority value
- **Preemption**: Allows scheduler to evict lower-priority Pods if a higher-priority one can't be scheduled.

### ⚖️ How Scheduling Works:

1. Scheduler places Pods in a priority queue.
2. It attempts to schedule higher-priority Pods first.
3. If no room is foundi preemption logic may evict lower-priority Pods.
4. ``preemptionPolicy: Never`` can disable eviction but still assign high scheduling order.


## [1] PriorityClass and Preemption Example

This example demonstrates how a high-priority Pod can evict lower-priority ones. The scenario uses a namespace with limited resources defined via a ResourceQuota, which will be covered later in the **Project Resources** section.

**YAML Files:**
- [ 01-test-priority-ns.yaml](./../k8s-files/pod-priority/01-test-priority-ns.yaml)
- [`02-priority-class.yaml`](./../k8s-files/pod-priority/02-priority-class.yaml)
- [`03-low-priority-pods.yaml`](./../k8s-files/pod-priority/03-low-priority-pods.yaml)
- [`04-high-priority-pod.yaml`](./../k8s-files/pod-priority/04-high-priority-pod.yaml)

#### ▶️ Hands-on Steps:

1. Create The Namespace and ``ResourceQuota``
    ```bash
    kubectl apply -f 02-predictable-demands/k8s-files/pod-priority/01-test-priority-ns.yaml
    
    # You can see the namespace quotas 
    kubectl describe quota priority-quota --namespace=priority-test
    ```

1. Create PriorityClass
    ```bash
    kubectl apply -f 02-predictable-demands/k8s-files/pod-priority/02-priority-class.yaml
    ```

2. Deploy Low-Priority Pods
    ```bash
    kubectl apply -f 02-predictable-demands/k8s-files/pod-priority/03-low-priority-pods.yaml
    kubectl get pods --namespace=priority-test
    ```

3. Attempt to Schedule High-Priority Pod
    ```bash
    kubectl apply -f 02-predictable-demands/k8s-files/pod-priority/04-high-priority-pod.yaml
    ```

4. Observe Preemption
    ```bash
    kubectl get events --namespace=priority-test --watch
    ```
    - You'll see 'preemption' events
    - A low-priority Pod will be terminated.
    - High-priority Pod transitions to 'Running'

5. Cleanup
    ```bash 
    #Delete the entire namespace to remove quota and pods
    kubectl delete namespace priority-test
    
    # Delete the cluster-wide PriorityClass
    kubectl delete priorityclass high-priority
    ```