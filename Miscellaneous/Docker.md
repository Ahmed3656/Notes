
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
- **Docker daemon (Dockerd)**: the server-side process that listens for Docker API requests, manages Docker objects (images, containers, networks, volumes), and communicates with the OS kernel.  
    (To check if it's running —> `ps -elf | grep dockerd`)
- **Containerd**: a container runtime that receives instructions from `dockerd` and handles container life cycle management, including pulling images, starting/stopping containers, and managing storage.  
    (To check if it's running —> `ps -elf | grep containerd`)
- **runc**: the low-level container runtime responsible for creating and running containers according to the OCI specification. It is invoked by Containerd to run each container.
- **Shim**: a lightweight process started by Containerd that sits between runc and the container. It keeps the container process alive after runc exits, captures logs, reports exit status, and allows daemon-less container management.  
    (You can see shim processes —> `ps -ef | grep shim`)

---

## Docker Network
Docker network is a virtual network layer managed by Docker that enables containers to communicate with each other, the host, or the internet through isolated, flexible, and customizable networking setups. For each running container on a network, Docker creates a virtual Ethernet (veth) adapter that pairs with a corresponding veth on the host or bridge, allowing containers to route traffic between each other efficiently.

#### Network Types
- **bridge** (The default network): creates an isolated internal network and gives containers private IPs to communicate while being isolated from the host. Use this when you want containers to communicate internally or expose ports manually.
- **host**: shares the host's networking stack directly with the container (no isolation), so it completely exposes the host to the container to where the container shares the host's hostname and has full access to its network configuration, ports, and adapters. Use this when you want the container to behave like a native process on your machine (e.g. low-latency or full access to host’s ports).
- **none**: Completely disables networking for the container. Use this for max isolation, or when the container doesn't need network access at all.
- **overlay**: creates a network that spans across multiple Docker hosts. Uses a built-in encrypted VXLAN to allow containers on different machines to communicate as if on the same network. Great for distributed systems and Docker Swarm.
- **macvlan**: assigns a MAC address to each container, making it appear as a physical device on the local LAN. Useful when containers need to be accessed directly from the external network like regular devices.

---
## Docker Volumes

A persistent storage mechanism managed by Docker that allows containers to store and share data independently from the container lifecycle. Volumes live outside the container filesystem, so data isn't lost when the container is stopped, removed, or rebuilt.

Before volumes, developers used bind mounts, where they used to manually link a host directory into the container. This tightly coupled the container to the host system, caused permission issues, and made it hard to manage or move containers. Volumes solved this by offering portable, isolated, and Docker-managed storage with better performance and consistency.

---

## Dockerfile
This is where we define the instructions for building a Docker image. Instead of running the base image and manually setting everything up, we can automate the build using a Dockerfile and run it with `docker build -t [account-name]/[image]:[tag]`. This approach is much cleaner, more consistent, and easier to maintain.

- FROM [repo]:[tag] (creates a new image) —> Specifies the base image to build upon. To start completely from scratch with no base image, use `FROM scratch`.
- WORKDIR [directory] (creates a new image) —> Sets the working directory inside the container; creates it if it doesn't exist. Equivalent to `mkdir` then `cd`.
- COPY [source content] [destination directory] (creates a new image) —> Copies files or directories from the host (from the build context directory) into the container at the specified path.
- ADD [url] [destination directory] (creates a new image) —> Downloads a file from a remote URL or copies local files (including automatic extraction of compressed archives like `.tar`) into the container. Similar to `COPY` but with extended functionality.
- SHELL ["path", "-c"] —> Overrides the default shell used to execute commands during the build process, allowing you to specify a custom shell (e.g. switching from `/bin/sh` to `/bin/bash`) for more control or compatibility.
- RUN [command arg1 arg2 ...](shell mode) **OR** ["command", "arg1", "arg2", ...](executive mode) (creates a new image) —> Executes a command during the image build process (build-time). It's very important to note that any command executed by RUN does not accept input during Dockerfile runtime.
- ENV [name=value] [name=value] ... (only affects metadata) —> Adds global environment variables to the image. You can add environment variables using `RUN export [name="value"]`, but this only affects the current shell session. The issue arises when running multiple `RUN` commands: each `RUN` creates a new intermediate container, and any variables set using `export` are lost when that temporary container ends. In contrast, `ENV` sets environment variables globally and persistently so they remain intact across layers and containers because they're baked into the image metadata.
- EXPOSE [port] (only affects metadata) —> Informs Docker that the container will listen on the specified network port at runtime.
- USER [user] (only affect metadata)  —> Switches the default user from root user to the specified user for any following instructions. Commonly used after creating a new user with `RUN groupadd [group] && useradd -g [group] [user]`, then applying `USER [user]` to improve security and avoid running processes as root.
- LABEL [key="value"] (only affects metadata)  —> Adds descriptive labels to the image in key-value format. Useful for organizing, filtering, searching, or documenting images with metadata like version, maintainer, or purpose.
- ENTRYPOINT ["path", "-c"] (only affects metadata) —> Defines the main command to run when a container starts (runtime). Unlike `CMD`, it cannot be overridden by arguments passed in `docker run` unless explicitly done with `--entrypoint`.
- CMD [command] (only affects metadata) —> Provides default arguments to the `ENTRYPOINT`, or acts as the main command if `ENTRYPOINT` is not set. It can be overridden by arguments passed in `docker run`.
- ARG [argument]=[value] —> Declares build-time variables used only during the image build process. Unlike `ENV`, `ARG` values are not preserved in the final image and can't be accessed at runtime.

<hr class="hr-light"/>

#### Important Notes
- If you want to base your image on a specific version of an image you've used before, you can use `FROM [repo]@[digest]` where the digest is the unique SHA256 hash of the image, ensuring you always build from the exact same version regardless of future updates to the tag.
- The `COPY` command accepts wildcards to match multiple files, e.g. `COPY *.py /app/`.
- If the filename or path contains spaces, use array syntax like `COPY ["name with spaces.ts", "/app"]`.
- You can add a `.dockerignore` file to exclude specific files or directories from being copied into the image during the build, keeping the image clean and lightweight.
- `RUN id` prints information about the current user inside the image, including user ID (UID), group ID (GID), and all associated groups for debugging user permissions during build.
- `ENTRYPOINT` and `CMD` are basically the same command, `ENTRYPOINT` is considered the command while `CMD` is considered the arguments of `ENTRYPOINT`

---

## Image Registries
Registries are used to store successfully built images so they can be easily pushed, pulled, shared, or deployed without rebuilding every time. Docker Hub is the default public registry, but you can also use private registries like GitHub Container Registry, AWS ECR, or host your own. Registries can store multiple tagged versions of the same image to help you manage updates and rollbacks efficiently.

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
- docker network disconnect [network] [name]  —> Disconnects host "name" from the network "network"

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
