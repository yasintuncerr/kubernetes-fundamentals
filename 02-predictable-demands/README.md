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
<br>

---

## Configuration Dependencies (ConfigMaps & Secrets)
Stateless applications often need external configuration to run correctly. Instead of hardcoding values like database endpoints or feature flags into a container image, Kubernetes provides dedicated resources to manage this data. This creates a runtime dependency: the application relies on these resources being present in the cluster to start and function correctly.

* ``ConfigMap``: Used to store non-confidental configuration data as key-value pairs.
* ``Secret``: Designed specifically for sensitive data like passwords, API tokens, or TLS certificates. Kubernetes handles Secrets more securely than ConfigMaps.

An application can consume this data in two primary ways: as **environment variables ot as **mounted files**.

---

* **``Configmap`` as Environment Variables**

    This is one of the most common methods for providing configuration to an application. You define your configuration data in a ``ConfigMap`` and then use ``valueFrom`` in your Pod specification to inject that data as environment variables into your container.

    ### Example
    
    In this example, we will:
    1. Create a ``ConfigMap`` named ``app-config`` that holds two configuration keys: ``APP_MODE`` and ``LOG_LEVEL``.
    2. Create a Pod that has a runtime dependency on this ``ConfigMap``.
    3. The Container inside the Pod will read these values from its environment variables and print them to standart output.

    You can find the YAML for this example here: [02-predictable-demands/k8s-files/03-configmap-env-vars.yaml](./k8s-files/03-configmap-env-vars.yaml)


    ### Hands-on Demonstration

    **1. Apply the Kubernetes Resources**
    Create the ``ConfigMap`` and the Pod by applying the manifest. If the ``ConfigMap`` does not exist, the Pod will fail to start.
    ```bash
    kubectl apply -f 02-predictable-demands/k8s-files/03-configmap-env-vars.yaml
    ```
    **2. Check the Pod Logs**
    Verify that the container has succesfully started and read the environment variables injected from the ``ConfigMap``.
    ```bash
    # Wait for the pod to be in 'Completed' state
    kubectl get pod configmap-env-pod

    # Check the logs
    kubectl logs configmap-env-pod
    ```
    You should see the output: ``Starting app in 'mod': production with log level: 'info'.`` 
    This confirms the application correctly consumed its configuration dependency

    **3. Cleanup**
    Remove the resources created in this example.
    ```bash
    kubectl delete -f 02-predictable-demands/k8s-files/03-configmap-env-vars.yaml
    ```
---


* **``Configmap`` as a Volume**

    Sometimes an application is desiged to read its configuration from files (e.g, ``app.properties``, ``settings.xml``). Kubernetes supports thsi by allowing you to mount a ``ConfigMap`` as a data volume. When you do this, each key in the ``ConfigMap``'s data field becomes a file in the mounted directory, and the value of that key becomes the content of the file.

    ### Example
    In this example we will:
    1. Create a ``ConfigMap`` containing two keys, each holding multi-line configuration data.
    2. Create a Pod that mounts this ``ConfigMap`` as a volume into the ``etc/config`` directory.
    3. The container will then list the files in that directory and print their contents, demonstrating its dependency on the ``ConfigMap`` for its configuration files.

    You can find the YAML for this example here: [02-predictable-demands/k8s-files/04-configmap-volume.yaml](./k8s-files/04-configmap-volume.yaml)


    ### Hands-on Demonstration
    **1. Apply the Kubernetes Resources**
    ```bash
    kubectl apply -f 02-predictable-demands/k8s-files/04-configmap-volume.yaml
    ```

    **2. Check the Pod Logs**
    
    The container's command is set up to explore the mounted volume and print what it finds.
    ```bash
    # Wait for the pod to be in 'Completed' state
    kubectl get pod configmap-volume-pod

    # Check the logs to see the file contents
    kubectl logs configmap-volume-pod
    ```
    The output will show the two files (``app.properties`` and ``user.settings``) that were created from the ``ConfigMap`` keys, along with their content. This provides the application succesfully used the volume as its configuration source.

    **3. Cleanup**
    ```bash
    kubectl delete -f 02-predictable-demands/k8s-files/04-configmap-volume.yaml
    ```






