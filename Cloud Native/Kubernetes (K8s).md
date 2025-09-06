Kubernetes is an open-source container orchestration platform designed to automate the deployment, scaling, and management of containerized applications. It operates at a richer level of abstraction than Docker Swarm not higher level as they both orchestrate containers but Kubernetes has much more powerful abstractions, treating entire data centers as a single, programmable entity. It is the de facto standard, with all major cloud providers (AWS EKS, Azure AKS, Google GKE) offering managed Kubernetes services.

#### The Core Problem Kubernetes Solves (Why K8s?)
Docker Swarm relied on individual containers as its building block. Containers are ephemeral and unreliable—they fail, die, and face problems frequently. Kubernetes was designed for **high availability, scalability, and disaster recovery**. Its core principle is that **everything must be loosely coupled**.

**The most crucial aspects Kubernetes aims to solve:**
- High Availability —> No matter what happens (a container crashes, a node dies, a zone fails), I always always need my application up and running. This is achieved by running multiple replicas across different failure domains and using self-healing mechanisms. **(Everything must be loosely coupled)**.
- Scalability —> The platform must be able to tolerate whatever load my application requires, both in terms of user traffic (horizontal pod autoscaling) and infrastructure needs (adding new worker nodes seamlessly).
- Disaster Recovery —> If the entire cluster completely collapses and fails, I should be able to recover my application state and configuration as fast as possible. This is achieved through declarative configuration stored in version control and persistent storage abstractions.

To achieve this, Kubernetes uses a more powerful and abstract building block: the **Pod**.

---

## Kubernetes Architecture
A Kubernetes cluster is made of **master (control plane) nodes** and **worker nodes**.
#### Manager (Control Plane) Node Components
- **API Server (`kube-apiserver`)** ≅ Docker Engine (Daemon) —> The Front Door / Brain Interface, this is the only component you (and all other components) talk to directly. It exposes the Kubernetes API, validates requests, and updates the state in `etcd`. It is the central management point.
- **`etcd`** —> A consistent and highly-available **key-value store** used as Kubernetes' backing store for **all cluster data**. It holds the entire desired and actual state of the cluster. Its role is identical to its role in Docker Swarm.
- **Scheduler (`kube-scheduler`)** —> Watches for newly created Pods that have no node assigned. It selects the **best node** for a Pod to run on based on resource requirements, constraints, and policies. It doesn't create the Pod itself; it just decides _where_ it should go.
- **Controller Manager (`kube-controller-manager`)** —> Runs **controller processes** that handle routine tasks. Each controller is a separate process, but they are compiled into a single binary.
    - **Node Controller:** handles checking if nodes go down.
    - **Replication Controller:** ensures the correct number of Pods are running for each ReplicaSet.
    - **Endpoints Controller:** populates the Endpoints object (links Services to Pods).
    - ...and many more. Its core job is to **watch the current state in `etcd` and constantly work to change the current state to match the desired state.**
- **Cloud Controller Manager (`cloud-controller-manager`)** —> An optional component that lets you link your cluster into your cloud provider's API. It contains controllers that are specific to the cloud provider (e.g., managing cloud load balancers, storage volumes, or network routes). If you're not on a cloud provider, you don't need this.

#### Worker Node Components
Worker nodes are the machines (VMs, physical servers) where your applications actually run.

- **Kubelet** —> An agent that runs on each worker node. Its job is to take a set of **PodSpecs** (Pod definitions) provided by the API server and ensure that the containers described in those PodSpecs are **running and healthy**. It's the node's supervisor, managing the container runtime and reporting back to the control plane.
- **Kube Proxy (`kube-proxy`)** —> A network proxy that runs on each node. It maintains network rules on the host, allowing communication to your Pods from inside or outside the cluster. It handles the complex networking and load-balancing for Services.
- **Container Runtime** —> The software responsible for running containers. **Kubernetes does not create containers itself.** It relies on a container runtime through the **Container Runtime Interface (CRI)**. This can be `containerd` (most common), `CRI-O`, Docker Engine, or others.

<hr class="hr-light" />

#### Key Objects & Abstractions
This is the higher-level thinking that solves Docker's container-level problems.

- **Pod** —> The smallest and simplest Kubernetes object. A Pod represents a **single instance of a running process** in your cluster. It's a logical "box" that encapsulates one or more application containers, shared storage (volumes), and a unique network IP. Containers within a Pod share the same lifecycle and network namespace (they can find each other via `localhost`).
- **ReplicaSet** —> Ensures that a specified number of identical Pod replicas are running at any given time. It's a pod supervisor. It creates and kills Pods to meet the replica count. You rarely manage these directly.
- **Deployment** —> A higher-level abstraction that **manages ReplicaSets and provides declarative updates for Pods.** You describe a desired state in a Deployment, and the Deployment Controller changes the actual state to the desired state at a controlled rate. This is how you achieve rolling updates and rollbacks. This is the object you will use most often for stateless applications.
- **Service** —> An abstraction that defines a logical set of Pods and a policy to access them. Pods are ephemeral—they get new IPs when they restart. A **Service** provides a stable, permanent IP address and DNS name that acts as a load balancer in front of a dynamic set of Pods. This is how Pods find and talk to each other.
- **Ingress** —> An API object that manages **external access** to the services in a cluster, typically HTTP/HTTPS. It provides features like load balancing, SSL termination, and name-based virtual hosting. An Ingress is not a Service; it sits in front of Services to route external traffic to them.
- **ConfigMap** —> An API object used to store non-confidential configuration data in key-value pairs. Pods can consume ConfigMaps as environment variables, command-line arguments, or configuration files in a volume. This lets you decouple environment-specific config from your container images.
- **PersistentVolume (PV)** —> Represents a piece of networked storage that has been provisioned in the cluster. It is a resource in the cluster, just like a node. Its lifecycle is independent of any individual Pod that uses it. This provides persistent storage that survives Pod restarts.
- **Namespace** —> Provides a mechanism for isolating groups of resources within a single cluster. Think of it as a virtual cluster inside your physical cluster. Useful for separating environments (dev, staging, prod) or teams.

---

## Important Notes
- **Manager nodes (the control plane) are typically only installed on Linux.**
- **Kubernetes does not create runtime containers directly.** It uses a container runtime (like `containerd` or `CRI-O`) through the CRI. This is a major difference from Docker Swarm, which managed containers directly.
- **Kubernetes has a rapid release cycle.** A new minor version is released approximately every **3 months**. Managed cloud services (EKS, AKS, GKE) handle these upgrades for you.
