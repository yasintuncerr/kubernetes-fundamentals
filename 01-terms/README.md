# **``Kubernetes``** Fundemantals

Kubernetes is an open-source container orchestration platform that automates the deployment, scaling, and management of containerized applications.

## Core Concepts/Primitives

* **``Containers``**: These are the building blocks for kubernetes-based-cloud-native applications. Container images are like classes, and containers are like objects, allowing for reuse and composition

* **``Pods``**: An atomic unit of scheduling, deployment, and runtime isolation for a group of containers. Containers within a Pod are always scheduled to the same host, deployed and scaled together, and can share resources like filesystems and networks. 

* **``Servicess``**: A simple yet powerful abstraction that provides a stable entry point (a permanent IP address and port number) for accesing an application, handling service discovery and load balancing for ephemeral Pods.

* **``Labels``**: Key-value pairs used to organize and select groups of resources, such as Pods, allowing them to be managed as logical units.

* **``NameSpaces``**: Provide a way to divide a Kubernetes cluster into logical pools of resources, offering scope for resource naming and a mechanism for applying authorization and policies.


## Extensibility 
Kubernetes is highly extensible through custom controllers and **CustomResourceDefinitions (CRDs)**, which allow users to add new domain-specific objects and automate complex application lifecycle management. This forms the basis of the "Operator" pattern.


## Workload Management
It supports various types of workloads, including:
* **Batch Jobs**: For short-lived, finite units of work that run reliably until completion
* **Prediodic Jobs(CronJobs)**: For executing units of work triggered by temporal events.
* **Daemon Services(DaemonSets)**: For running prioritized, infrastructure-focused Pods on specific nodes, often for cluster-wide operations like logging or monitoring.
* **Stateless Services**: Applications composed of identical, ephemeral replicas that can be rapidly scaled and made highly available.
* **Stateful Services**: Applications that require persistent identity, networking, storage, and ordinality, managed by StatefulSets.


## Security:
Kubernetes provides various security features, including process containment (limiting priviligies), network segmentation (restricting traffic), secure configuratiıon (managing sensitive data), and access control(authentication and authorization to the API server).

## Scaling
Kubernetes supports elastic scaling in multiple dimensions. 
* **Horizontal Pod AutoScaling (HPA)**: Adjusting the number of Pod replicas based on load.
* **Vertical Pod AutoScaling (VPA)**: Adapting resource requirements (CPU, memory) for Pods.
* **Cluster AutoScaling (CA)**: Changing the number of cluster nodes to meet demand.

## Image Building 
Kubernetes can also be used for building container images directly within the cluster, offering benefits like reduced maintenance costs and simplified transitions between build and run phases.

