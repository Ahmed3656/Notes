
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

<hr class="hr-light"/>

#### Control Groups (cgroups)
Control Groups are a feature in the Linux kernel used to control and limit how much system resources a group of processes can use. They restrict containers in terms of CPU usage, memory, disk I/O, and more. They also help prevent a single container from consuming too many resources and affecting the rest of the system.

They work by grouping processes and applying resource rules to the group as a whole. For example, a container can be limited to 512MB of memory or 25% of the CPU, if it exceeds that limit, the kernel can stop it, throttle it, or prevent it from using more. Cgroups can also monitor how much a container is using in real time.

This makes cgroups important for keeping containers efficient and stable, especially when running multiple containers on the same host.

<hr class="hr-light"/>

#### Very Important Notes
- When a container runs, it does so solely for the main process (PID 1) it was started with. If that process exits or is killed, the container stops and all associated processes are terminated.
- If a newly downloaded image shares layers with existing images, Docker reuses the existing layers and only downloads the new ones, instead of fetching the entire image again.
- Images are typically small because they are designed to serve as a minimal base environment for the application.
- Image name = [repo]:[tag] where the repo is the application's name I want to use and the tag is the version. (ex. python:3.10 | ubuntu:20.04)

---

## Docker Engine
- **Docker client**: the user-facing CLI tool used to run Docker commands and interact with Docker components.
- **Docker daemon (dockerd)**: the server-side process that listens for Docker API requests, manages Docker objects (images, containers, networks, volumes), and communicates with the OS kernel.  
    (To check if it's running —> `ps -elf | grep dockerd`)
- **Containerd**: a container runtime that receives instructions from `dockerd` and handles container life cycle management, including pulling images, starting/stopping containers, and managing storage.  
    (To check if it's running —> `ps -elf | grep containerd`)
- **runc**: the low-level container runtime responsible for creating and running containers according to the OCI specification. It is invoked by Containerd to run each container.
- **Shim**: a lightweight process started by Containerd that sits between runc and the container. It keeps the container process alive after runc exits, captures logs, reports exit status, and allows daemon-less container management.  
    (You can see shim processes —> `ps -ef | grep shim`)

---

## Docker Commands

#### Container & image Commands
- docker pull [image] —> Download an image from Docker Hub
- docker create [image] —> Create a new container but do not start it
- docker start [container] —> Start a stopped container
- docker run [image] —> Run a container from an image (same as pull then create then start)
- docker build -t [name] . —> Build an image from a Dockerfile
- docker tag [image] [repo:tag] —> Tag an image with a custom name
- docker ps or docker container ls —> List running containers
- docker ps -a or docker container ls -a —> List all containers including stopped ones
- docker images —> List downloaded Docker images
- docker inspect [image] —> View details of the image (in JSON)
- docker stop [container] —> Gracefully stop a running container
- docker kill [container] —> Force-stop a container immediately
- docker restart [container] —> Restart a container
- docker rm [container] —> Remove a container (delete)
- docker rmi [image] —> Remove an image
- docker exec -it [container] bash —> Open shell in a running container
- docker logs [container] —> View logs of a container

<hr class="hr-light"/>

#### Networking / Ports
- docker run -p 8080:80 [image] —> Map host port 8080 to container port 80
- docker network ls —> List all Docker networks
- docker network inspect [network] —> View details of a specific network
- docker network create [name] —> Create a custom Docker network

<hr class="hr-light"/>

#### Volumes / Data Management
- docker volume ls —> List all volumes
- docker volume create [name] —> Create a new volume
- docker run -v [volume]:/data [image] —> Mount a volume into a container
- docker inspect [container] —> Show detailed info about a container

<hr class="hr-light"/>

#### System Cleanup
- docker system prune —> Remove all unused data (containers, images, networks, etc.)
- docker image prune —> Remove unused images
- docker volume prune —> Remove unused volumes

<hr class="hr-light"/>

#### Important Notes
- Adding -it in the run command allows you to run the container or image in interactive mode, where `-i` stands for interactive (keeps STDIN open so you can type commands), and `-t` allocates a pseudo-TTY (gives you a terminal interface with proper formatting), enabling you to interact directly with the container as if you're inside a shell.
- Adding -d in the run command runs the container in detached mode, meaning it runs in the background without attaching to your terminal. This allows the container to continue running independently of your session, and Docker will print the container ID so you can manage it later using commands like `docker logs`, `docker stop`, or `docker exec`. Detached mode is ideal for long-running services or daemons.
- You can configure the container's restart policy by adding `--restart` to the `docker run` command, followed by one of the available options: `always` (restarts the container if the main process exits or if the Docker daemon restarts), `unless-stopped`(restarts if the main process is killed but doesn't if Docker daemon restarts), - or `on-failure` (restarts only if the process exits with a non-zero status code or if the Docker daemon restarts). All policies don't restart the container if you manually stop it.
- Commands executed during the creation or setup of a container are called **build-time commands**, while those executed while the container is running are referred to as **runtime commands**.
- Docker allows you to pull official images directly, as they are published and maintained by the official organizations. To pull an unofficial image created by an individual developer, you must specify the account name before the image name, e.g., `docker image pull [account-name]/[image-name]`.

---
