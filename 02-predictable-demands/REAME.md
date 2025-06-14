# Kubetnetes Patterns: Predictable Demands Examples
This repository contains a hands-on example demontsrating the "Predictable Demands" pattern, as described in Chapter 2 of **Kubernetes Patterns book**. The focus is on declaring an application's runtime dependencies.

## Understanding Runtime Dependencies
For Kubernetes to effectively manage and scedule your applications, it needs to understand their **runtime dependencies**. If these declared needs aren't met your application might fail to schedule or start correctly. The two most common types of runtime dependencies are **storage** and **configuration**

## Storage Dependencies(Volumes)
Containers have ephermal filesystems, meaning any data written inside a container is lost when it restarts. To persist data or share it between containers, Kubernetes uses **Volumes**. A Pod decleras a dependency on a certain type of storage, and the scheduler uses this information to place the Pod on a node that can satisfy the requirement.

* ## ``emptyDir``
    A simple, temporary volume that shares the Pod's lifecycle. It's perfect for sharing files between containers within the same Pod but is deleted when the Pod is removed.


    
    ### Example  
    This example demonstrates a Pod with two containers sharing data using an ``emptyDir`` volume:

    * A **writer-container** writes a log file to the shared volume every 5 seconds.
    * A **reader-container** reads from this log file every 5 seconds.

    You can find the YAML file for this example her: [02-predictable-demands/k8s-files/01-empty-dir-volume.yaml](./k8s-files/01-empty-dir-volume.yaml)


    ### Hands-On Demonstration
    
    **1. Create the Pod**

    Apply the configuration to create the Pod and its associated ``emptyDir`` volume.
    ```bash
    kubectl apply -f 02-predictable-demands/k8s-files/01-empty-dir-volume.yaml
    ```
    **2. Check the Pod Status**
    
    Verify that the ``empty-dir-pod`` is running succesfully.
    ```bash
    kubectl get pods
    ```

    **3. View Container Logs**

    You can inspect the standart output of each container to see them interacting with the shared volume.
    ```bash
    # for writer
    kubectl logs empty-dir-pod -c writer-container
    
    
    # for reader
    kubectl logs empty-dir-pod -c reader-container
    ``` 

    **4. Test Data Persistence on Container Restart**

    If you were to restart one of the containers, you would observe that the log file within the ``emptyDir`` volume **persists**, and the containers continue their read/write operations without data loss

    ```bash
    # restart writer-container
    kubectl exec empty-dir-pod -c writer-container -- kill 1

    # restart reader-container
    kubectl exec empty-dir-pod -c reader-container -- kill 1
    ```
    **5. Test Data Loss on Pod Deletion**

    Now, let's see what happens when entire Pod is removed and recreated.
    ```bash
    # Delete The Pod
    kubectl delete pod empty-dir-pod

    ## Recreate the Pod
    kubectl apply -f 02-predictable-demands/k8s-files/01-empty-dir-volume.yaml
    ```
    When you inspect the logs again you'll notice that original log files is gone. The **writer-container** has started a new log file in fresh ``emptyDir`` volume because the old volume was destroyed along with the original Pod. This demonstrates the ephemeral nature of ``emptyDir`` volumes.
    
    ***

* ## ``PersistentVolumeClaim``
    This example expands on the concept of **Predictable Demands** by demonstrating how applications can be request durable, persistent storage that exists beyond lifecycle of a single Pod. This achieved using ``PersistanceVolume``(PV) and ``PersitentVolumeClaim``(PVC) objects in Kubernetes. 

    As we saw with ``emptyDir``, some volumes are tied to the Pod's lifecycle. When the Pod is deleted the data is lost. For stateful applications like databases, file servers, or applications that need to maintain state across restarts and rescheduling, this is not a viable solution.

    Kubernetes solves this with a powerful abstraction: ``PersistentVolume`` and ``PersistentVolumeClaim``.

    * ``PersistentVolume``(PV): A piece of storage in the cluster that has beeen provisioned by an administrator. It is a cluster resource, just like a CPU or memory. A PV represents the "supply" of storage.
    * ``PersistentVolumeClaim``(PVC): A request for storage by a user or application. It is similiar to how a Pod consumes Node resources (CPU/memory). A PVC represents the "demand" for storage.

    This model decouples the application's need for storage from the underlying storage technology. The application simply requests storage, and Kubernetes finds a suitable provisioned volume to fullfill that request.

    ### Example  
    In this eample, we will create a complete persistent storage workflow:

    1. A ``PersistentVolume``(PV) provides 1Gi of storage using directory on the hosts node's filesystem(``hostPath``).
    2. A ``PersistentVolumeClaim``(PVC) requests 500Mi of that storage. Kubernetes will bind this claim to our PV.
    3. A Pod mount the volume via the PVC and runs a container that writes the current date to a file every 5 seconds.

    This demonstrates that even if the Pod is deleted and recreated, the data written to the file will persist.

    You can find the YAML for this example here: [02-predictable-demands/k8s-files/02-persistent-volume-claim.yaml](./k8s-files/02-persistent-volume-claim.yaml)


        ⚠️ Note on hostPath: This example uses a hostPath volume, which maps a directory from the hosting Node directly into your Pod. This is suitable for single-node clusters like Minikube or for development purposes. In a real multi-node production cluster, you would use a network storage provider (e.g., GCEPersistentDisk, AWSElasticBlockStore, NFS) to ensure the storage is accessible from any node the Pod might be scheduled on.


    ### Hands-on Demonstration
    Since we are using ``hostPath``, you first create the directory on your Kubernetes node. If your desired ``hostPath`` requiered permission first set it.

    **1. Apply the Kubernets Resources**
    
    Create the PV, PVC, and the Pod all at once by applying the manifest file.
    ```bash
    kubectl apply -f 02-predictable-demands/k8s-files/02-persistent-volume-claim.yaml
    ```

    **2. Check the status of PV and PVC**
    
    Verify that the PVC has succesfully "bound" to the PV.

    ```bash
    #Check the Persistent Volume
    kubectl get pv my-pv-storage

    # Check the Persistent Volume Claim
    kubectl get pvc my-pvc-claim
    ```
    You should see that **STATUS** for both resources is ``Bound``

    **4. Verify Application is Writing Data**
    
    Check the Pod's status and than look inside the container to see the data file being written.
    ```bash

    #Check the pod is running
    kubectl get pod persistent-storage-pod

    # Peek inside the pod to see the content of the file
    kubectl exec persistent-storage-pod --cat /app-data/status.txt
    ```
    You should see a line with the date and time. Wait a few seconds and run the ``exec`` command again to see it update.

    **5 Test Persistence Across Pod Deletion**
    This is the key test. We will delete the Pod and then recreate it The data in /app-data/status.txt should survive this process.

    * **First Delete te Pod**
    
        ```bash
        kubectl delete pod persistent-storage-pod
        ```
        Wait a few moments for the Pod to be terminated. The PV and PVC are not affected by this.

    * **Next Recreate the Pod** 

        ``` bash 
        kubectl apply -f 02-predictable-demands/k8s-files/02-persistent-volume-claim.yaml
        ```
    * **Finally verify the data is still there**
    
        Check the contents of the file in the new Pod
        ```bash
        kubectl exec persistent-storage-pod -- cat /app-data/status.txt
        ```

    You should see the **last date and time written by the original Pod**, proving that the data has persisted. Tge new Pod will now resume updating the file every 5 seconds.


    **6. Cleanup**
    To remove all the resources created in this example(including PV and PVC), you can delete them using same file.
    ```bash
    kubectl delete -f 02-predictable-demands/k8s-files/02-persistent-volume-claim.yaml
    ```

