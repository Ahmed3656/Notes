
## Why Docker?
People used to run applications on multiple virtual machines. However, issues arise when deploying multiple application components across different VMs, as each VM requires its own operating system and other major components. This leads to significant resource waste, especially since the primary need is just the application and its dependencies.

Normally, virtual machines consist of a virtual hardware layer, followed by an operating system layer, then the application’s dependencies, and finally the main application itself. This results in multiple kernels running across VMs. Containerization, and tools like Docker, address this by allowing all components to run on a single shared kernel, eliminating the need for a separate OS instance for each application.

---
## Containers
Containers are lightweight, isolated environments that run on the same operating system kernel as the host machine. They build on top of the host kernel by adding a minimal OS layer, followed by additional layers for dependencies based on the application's requirements, and finally the application itself. When a change is made to the container's configuration, a new layer is added, similar to a new referencing disk (similar to how VM snapshots work), preserving the previous layers and enabling efficient versioning and storage.

Once a container is configured with the necessary files and dependencies, and is ready for use, it becomes a reusable template known as an **image**. The process of creating this image from a configured container is called **building** the image. When an image is retrieved for use, the process is referred to as **pulling** the image, and creating a running instance from it is called **creating** or **running** a container from the image.

<hr class="hr-light"/>
#### Namespaces

Namespaces are a feature in the Linux kernel that isolate resources for processes. They allow each container to believe it’s running on its own system, even though it shares the host kernel. Namespaces isolate things like process IDs, hostnames, users, and network interfaces, so that each container has its own separate view of the system.

There are different types of namespaces that isolate different parts of the system:

- PID namespace separates process trees
- NET namespace isolates network interfaces and routing
- MNT namespace isolates file system mount points
- UTS namespace isolates host and domain names
- IPC namespace separates interprocess communication
- USER namespace separates user and group IDs

Together, these create the illusion that each container is independent from the rest, even though they’re running on the same OS.

<hr class="hr-light">

##### Control Groups (cgroups)
Control Groups are a feature in the Linux kernel used to control and limit how much system resources a group of processes can use. They restrict containers in terms of CPU usage, memory, disk I/O, and more. They also help prevent a single container from consuming too many resources and affecting the rest of the system.

They work by grouping processes and applying resource rules to the group as a whole. For example, a container can be limited to 512MB of memory or 25% of the CPU, if it exceeds that limit, the kernel can stop it, throttle it, or prevent it from using more. Cgroups can also monitor how much a container is using in real time.

This makes cgroups important for keeping containers efficient and stable, especially when running multiple containers on the same host.

---

