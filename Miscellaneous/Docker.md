
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

This is where we define the instructions for building a Docker image. Instead of running the base image and manually setting everything up, we can automate the build using a Dockerfile and run it with `docker build -t <account-name>/<image>:<tag>`. This approach is much cleaner, more consistent, and easier to maintain.

- `FROM <image>[:<tag>]` (creates a new layer) —> Specifies the base image to build upon. To start completely from scratch with no base image, use `FROM scratch`.
- `WORKDIR <directory>` (creates a new layer) —> Sets the working directory inside the container; creates it if it doesn't exist. Equivalent to `mkdir` then `cd`.
- `COPY <src> <dest>` (creates a new layer) —> Copies files or directories from the host (from the build context) into the container at the specified path.
- `ADD <src> <dest>` (creates a new layer) —> Downloads a file from a remote URL or copies local files (including automatic extraction of compressed archives like `.tar`) into the container. Similar to `COPY` but with extended functionality.
- `SHELL ["<path>", "<arg>", ...]` —> Overrides the default shell used to execute commands during the build process, allowing you to specify a custom shell (e.g. switching from `/bin/sh` to `/bin/bash`) for more control or compatibility.
- `RUN <command> <arg1> <arg2> ...` (shell form) **OR** `["<executable>", "<arg1>", "<arg2>", ...]` (exec form) (creates a new layer) —> Executes a command during the image build process (build-time). It's very important to note that any command executed by `RUN` does not accept input during Dockerfile runtime.
- `ENV <name>=<value> [<name>=<value> ...]` (metadata) —> Adds global environment variables to the image. You can add environment variables using `RUN export <name>="<value>"`, but this only affects the current shell session. The issue arises when running multiple `RUN` commands: each `RUN` creates a new intermediate container, and any variables set using `export` are lost when that temporary container ends. In contrast, `ENV` sets environment variables globally and persistently so they remain intact across layers and containers because they're baked into the image metadata.
- `EXPOSE <port> [<port>/<protocol>...]` (metadata) —> Informs Docker that the container will listen on the specified network port(s) at runtime. The protocol (tcp/udp) is optional and defaults to tcp.
- `USER <user>[:<group>]` (metadata) —> Switches the default user from root to the specified user for any following instructions. Commonly used after creating a new user with `RUN groupadd <group> && useradd -g <group> <user>`, then applying `USER <user>` to improve security and avoid running processes as root.
- `LABEL <key>=<value> [<key>=<value> ...]` (metadata) —> Adds descriptive labels to the image in key-value format. Useful for organizing, filtering, searching, or documenting images with metadata like version, maintainer, or purpose.
- `ENTRYPOINT ["<executable>", "<param>", ...]` (metadata) —> Defines the main command to run when a container starts (runtime). Unlike `CMD`, it is not easily overridden by arguments passed in `docker run` unless explicitly done with `--entrypoint`.
- `CMD ["<executable>", "<param>", ...]` (metadata) —> Provides default arguments to the `ENTRYPOINT`, or acts as the main command if `ENTRYPOINT` is not set. It can be overridden by arguments passed in `docker run`.
- `ARG <name>[=<default value>]` —> Declares build-time variables used only during the image build process. Unlike `ENV`, `ARG` values are not preserved in the final image and can't be accessed at runtime. Values can be passed via the `--build-arg <name>=<value>` flag on `docker build`. 

<hr class="hr-light"/>

#### Important Notes

- If you want to base your image on a specific version of an image you've used before, you can use `FROM <image>@<digest>` where the digest is the unique SHA256 hash of the image, ensuring you always build from the exact same version regardless of future updates to the tag.    
- The `COPY` instruction accepts wildcards to match multiple files, e.g., `COPY *.py /app/`.
- If the filename or path contains spaces, use the exec form (JSON array) syntax like `COPY ["file with spaces.txt", "/app"]`.
- You can add a `.dockerignore` file to exclude specific files or directories from being copied into the image during the build, keeping the image clean and lightweight.
- `RUN id` prints information about the current user inside the image, including user ID (UID), group ID (GID), and all associated groups for debugging user permissions during build.
- `ENTRYPOINT` and `CMD` work together. `ENTRYPOINT` is typically set to the main executable, while `CMD` provides the default arguments for that executable. The full command executed at runtime is `<ENTRYPOINT> <CMD>`.

---

## Image Registries
Registries are used to store successfully built images so they can be easily pushed, pulled, shared, or deployed without rebuilding every time. Docker Hub is the default public registry, but you can also use private registries like GitHub Container Registry, AWS ECR, or host your own. Registries can store multiple tagged versions of the same image to help you manage updates and rollbacks efficiently.

---

## Docker Compose
Docker Compose is a tool that helps you define, configure, and run **multi-container Docker applications** using a single YAML file. Instead of running individual containers one by one with long `docker run` commands, Compose allows you to orchestrate them all by only running `docker-compose up`.  
This is especially useful when your application is made of multiple components like a frontend, backend, database, cache, and reverse proxy. Each component becomes its own **service**, and Compose takes care of how they communicate, their network, volumes, environment variables, and even build steps.

<hr class="hr-light"/>

#### The heart of Compose is the `docker-compose.yml` file, where you define:
- Which **services** you want to run
- How to build or pull their **images**
- What **ports**, **volumes**, and **networks** they need
- Whether they should restart automatically
- Their **environment variables**, **secrets**, and **configs**
- Their dependencies and startup order

There are four main keys:
- **`version`** (required) —> Specifies which version of the Compose file syntax you’re using. This helps Docker understand how to interpret the structure and features in your file. (Note: The `version` key is largely obsolete in newer Compose versions that use the `compose.yaml` name and prioritize the `services` top-level key).
- **`services`** (required) —> Defines each container that makes up your app. You describe how to build or pull the image, expose ports, link dependencies, set environment variables, mount volumes, and more. Each service acts like one unit of your architecture (e.g. `web`, `db`, `auth-service`).
- **`networks`** (optional) —> Tells Docker to create custom virtual networks that your containers will use to communicate. This lets you isolate traffic between services and assign fine-grained control over how they connect.
- **`volumes`** (optional) —> Instructs Docker to create persistent storage volumes that live outside the container lifecycle. These volumes can be shared across services and reused even after containers are removed or rebuilt.

<hr class="hr-light"/>

#### Common Service Configuration Keys:
- **`build`** —> Specifies the path to the Dockerfile directory for building an image instead of pulling a pre-built one.
- **`image`** —> Defines the image to use for the container (e.g., `nginx:alpine`).
- **`ports`** —> Maps host ports to container ports (e.g., `"8080:80"`).
- **`expose`** —> Exposes ports to other services on the same network without publishing them to the host.
- **`environment`** —> Sets environment variables inside the container (can use a list or a key-value map).
- **`env_file`** —> Specifies a file to read environment variables from.
- **`volumes`** —> Mounts host paths or named volumes into the container.
- **`networks`** —> Attaches the service to specific custom networks.
- **`depends_on`** —> Defines startup dependencies between services (e.g., the `app` service depends on the `db` service).
- **`restart`** —> Defines the container's restart policy (e.g., `always`, `unless-stopped`, `on-failure`).
- **`deploy`** —> Specifies configuration related to deploying the service in Swarm mode (e.g., replicas, update config, resources). This key is ignored when using `docker-compose up`.

<hr class="hr-light" />

#### Services VS Microservices

- **A Service (in Docker Compose context):** refers to a single containerized component of your application defined in the `docker-compose.yml` file. It is a technical unit of orchestration. A "web" service might be an entire monolithic application.
- **A Microservice:** is an architectural and organizational concept. It is a small, independent, loosely-coupled service that implements a specific business capability (e.g., "user authentication service", "payment processing service").
- **The Relationship:** A single **microservice** is typically deployed as a Docker Compose **service**. A complete microservices architecture is therefore orchestrated by a `docker-compose.yml` file that defines many **services**, each representing a different **microservice**. Compose simplifies the local development and testing of a multi-**microservice** application by allowing you to manage all its constituent **services** together.

---

## Docker Swarm
Docker Swarm is Docker's native clustering and orchestration tool. It creates a cluster of Docker hosts (called nodes) which act as a single virtual system. Swarm orchestrates services across these nodes, handling deployment, scaling, desired state reconciliation, load balancing, and networking.

**Key Concepts:**
- **Node:** A single machine (physical or virtual) running Docker that participates in the swarm. A node is either a **manager** or a **worker**.
- **Manager Node:** Handles cluster management tasks, maintains the cluster state using the embedded **etcd database**, dispatches tasks to worker nodes, and serves the swarm mode HTTP API. **Only manager nodes can execute commands to inspect or modify the cluster state.**
- **Worker Node:** Receives and executes tasks (containers) dispatched from manager nodes.
- **Service:** A definition of the tasks to execute on the nodes (e.g., which image to run, how many replicas, published ports). It is the central structure of the swarm system.
- **Task:** A running container that is part of a service. A service's "replicas" setting defines how many identical tasks should be running.

**etcd database:** etcd is a distributed, reliable key-value store designed for holding critical data that must be available to all members of a distributed cluster. In the Docker world, it is the default and underlying database that powers Docker Swarm, storing and managing the state of the entire cluster (service definitions, network configs, secrets, etc.).

**Converge:** The process by which the swarm manager nodes continuously compare the **actual state** of the running tasks against the **desired state** defined in the service. If they do not match (e.g., a task crashes), the manager takes action to reconcile them by starting new tasks to meet the desired replica count.

<hr class="hr-light" />

**To Initialize the Docker Swarm:** run `docker swarm init --advertise-addr <ip:port> --listen-addr <ip:port>`. This command:
- Creates the embedded etcd database.
- Converts the current Docker daemon into the first manager node (the Leader).
- Generates two unique join tokens (one for workers, one for managers) for security.

**We can have as many manager nodes as we want but the recommended range is 1 to 7 and it must be an odd number (1, 3, 5, 7)** to maintain a healthy quorum for the Raft consensus algorithm, which prevents split-brain scenarios in the etcd database.

**To Add a Node to the Swarm:** On an existing manager, run `docker swarm join-token worker` to get the command to join as a worker or `docker swarm join-token manager` to join as a manager. Copy the provided command and run it on the node you want to add.

**To Check the Nodes in the Swarm:** run `docker node ls`. This command only works on manager nodes.

**To Create a Service:** run `docker service create --name <name> -p <published-port>:<target-port> --replicas <amount> <image>`. This defines the desired state for the service, and the swarm manager will immediately work to **converge** the actual state to match it by scheduling tasks across available nodes.

**To View the Services:** run `docker service ls`

**To Inspect a Service:** run `docker service inspect <service-name>`

**To View the Tasks of a Service:** run `docker service ps <service-name>`

**To Scale a Service:** run `docker service scale <name>=<amount>`. This updates the desired state (the number of replicas), and the swarm will **converge** by starting or stopping tasks to match the new count.

**To Update a Service:** run `docker service update --image <new-image> <service-name>` to change the image, or use flags to update other parameters like environment variables, replicas, or resources.

**To Remove a Service:** run `docker service rm <service-name>`

<hr class="hr-light" />

#### Important Note: Services Require Long-Lived Processes

A fundamental Docker Swarm requirement is that a service must run a **persistent process**. The health of a container is determined by its main process (PID 1). If this process exits, the container is considered failed.
- **Why Services Fail:** swarm expects a service to be a continuous workload (e.g., a web server, database, API). If the container's command is a short-lived utility, script, or shell that completes its task and terminates, the container stops. Swarm's convergence loop detects this failure and continuously attempts to restart it, creating a **crash loop**.
- **This is Not Image-Specific:** the issue is not the image's origin (OS or application) but the **command it runs**. An `nginx` image works because its default command launches the Nginx server daemon. A `python` image will fail unless its command is overridden to run a persistent Python application (e.g., `python app.py`).
- **The Solution:** always ensure the image and command used for a service are designed to run indefinitely. For images meant for one-time tasks, you must override the command to a persistent one (e.g., `sleep infinity`) for testing, but this is not suitable for production services.
- **Key Takeaway:** swarm orchestrates **services**, not **tasks**. A service is defined by an ongoing process. If your container's natural state is "exited," it is not a service and will not run correctly in Swarm mode.

**GUI for the Swarm Nodes:** you can deploy a visualizer tool as a service within your swarm (e.g., `docker service create --name=visualizer ...`). Once running, you can open `http://localhost:<port>` or the node's IP to see a GUI representation of the nodes and how tasks are distributed.

---

## Docker Stack
Docker Stack is a Swarm-mode command that deploys and manages a complete application stack, defined in a Compose file, across a cluster of Docker hosts. While `docker-compose` orchestrates multi-container applications on a single host, `docker stack deploy` orchestrates multi-service applications across an entire swarm. It translates your Compose file into Swarm's native objects: services, overlay networks, configs, and secrets.

This is the primary method for deploying production-ready, scalable, and self-healing applications to a Docker Swarm.

<hr class="hr-light"/>

#### The heart of Stack is the `docker-stack.yml` file, where you define:
- Which **services** you want to run and how to scale them
- What **images** to use (must be pre-built in a registry)
- What **ports** to publish through the swarm's routing mesh
- What **volumes** and **networks** to create as swarm objects
- Their **environment variables**, **secrets**, and **configs**
- Their resource constraints, placement preferences, and update strategies 

There are four main keys (inherited from Compose, with Swarm-specific behavior):
- **`services`** (required) —> Defines each service that makes up your app. In Swarm, a service is a template for tasks (containers) that will be distributed across multiple nodes. The `deploy:` sub-key is crucial here for Swarm-specific configuration.
- **`networks`** (optional) —> Tells Docker to create **overlay networks** by default. These virtual networks span all nodes in the swarm, enabling seamless communication between containers on different hosts. This key is no longer optional for multi-host communication.
- **`volumes`** (optional) —> Instructs Docker to create volumes that can be used by services across the swarm. The actual persistence depends on the volume driver used (e.g., `local` for node-specific storage, or cloud drivers for shared storage).
- **`configs`** & **`secrets`** (optional) —> Define configuration files and sensitive data that will be securely injected into the service's containers at runtime. These are Swarm-native objects that are essential for production configuration management. 

<hr class="hr-light"/>

#### Essential `deploy:` Configuration Keys (Swarm-Specific):

The `deploy:` key is ignored by `docker-compose` but is critical for `docker stack deploy` to define the service's orchestration rules.
- **`replicas`** —> Defines the number of identical tasks (containers) to run for the service. The swarm will distribute these across available nodes.
- **`update_config`** —> Configures how the service is updated without downtime (rolling update). 
    - `parallelism`: Number of containers to update at once.
    - `delay`: Time to wait between updating batches of containers.
    - `order`: Update order (`start-first` for zero-downtime).    
- **`rollback_config`** —> Configures how to automatically revert a failed update.
- **`restart_policy`** —> Defines if/when to restart failed containers (e.g., `condition: on-failure`).
- **`placement`** —> Specifies constraints for which nodes can run the service (e.g., `constraints: ['node.role==manager']`).
- **`resources`** —> Sets limits and reservations for CPU and memory usage to manage node resources effectively.

---

## Docker Commands

#### Container & image Commands
- `docker pull <image>` —> Download an image from a registry
- `docker create <image>` —> Create a new container but do not start it
- `docker start <container>` —> Start a stopped container
- `docker run <image>` —> Run a container from an image (same as pull then create then start)
- `docker build -t <name> .` —> Build an image from a Dockerfile
- `docker tag <image> <repo:tag>` —> Tag an image with a custom name
- `docker ps` or `docker container ls` —> List running containers
- `docker ps -a` or `docker container ls -a` —> List all containers including stopped ones
- `docker images` or `docker image ls` —> List downloaded Docker images
- `docker inspect <object-name>` —> View detailed low-level information (in JSON) of an image, container, network, or volume.
- `docker stop <container>` —> Gracefully stop a running container
- `docker kill <container>` —> Force-stop a container immediately
- `docker restart <container>` —> Restart a container
- `docker rm <container>` —> Remove a stopped container (delete)
- `docker rmi <image>` —> Remove an image
- `docker exec -it <container> <command>` —> Execute a command in a running container (e.g., `bash`, `sh`)
- `docker logs <container>` —> View logs of a container
- `docker logs -f <container>` —> Follow (tail) the logs of a container in real-time

<hr class="hr-light"/>

#### Networking
- `docker run -p <host-port>:<container-port> <image>` —> Map host port to container port
- `docker network ls` —> List all Docker networks
- `docker network inspect <network>` —> View details of a specific network
- `docker network create <name>` —> Create a custom Docker network
- `docker network connect <network> <container>` —> Connect a container to a network
- `docker network disconnect <network> <container>` —> Disconnect a container from a network
- `docker network prune` —> Remove all unused networks 

<hr class="hr-light"/>

#### Volumes / Data Management
- `docker volume ls` —> List all volumes
- `docker volume create <name>` —> Create a new volume
- `docker run -v <volume-name>:<container-path> <image>` —> Mount a named volume into a container
- `docker run -v <host-path>:<container-path> <image>` —> Mount a host directory into a container (bind mount)
- `docker volume inspect <volume-name>` —> Show detailed info about a volume
- `docker volume prune` —> Remove all unused volumes 

<hr class="hr-light"/>
<hr class="hr-light"/>

#### Compose Commands
- `docker-compose up` —> Build, create, start, and attach to containers for all services.
- `docker-compose up -d` —> Start all services in detached mode.
- `docker-compose down` —> Stop and remove all containers, networks, and volumes defined in the compose file.
- `docker-compose ps` —> List the containers for the current compose project.
- `docker-compose logs [service]` —> View output from the specified service's containers.
- `docker-compose logs -f [service]` —> Follow the logs for a service.
- `docker-compose build` —> Build or rebuild images for services defined with a `build` key.
- `docker-compose pull` —> Pull the latest images for services defined with an `image` key.
- `docker-compose config` —> Validate and view the compiled compose configuration. 

<hr class="hr-light"/>

#### Swarm Commands
- `docker swarm init [--advertise-addr <ip>]` —> Initialize a swarm and make the current node a manager.
- `docker swarm join-token <manager|worker>` —> Display the command and token for joining a new node.
- `docker swarm leave [--force]` —> Remove the current node from the swarm (use `--force` on a manager).
- `docker node ls` —> List all nodes in the swarm (manager command).
- `docker node ps [node-name]` —> List tasks running on a specific node.
- `docker node promote <node-name>` —> Promote a worker node to a manager.
- `docker node demote <node-name>` —> Demote a manager node to a worker.
- `docker node update --availability drain <node-name>` —> Drain a node, moving its tasks to other nodes for maintenance.
- `docker service create [options] <image>` —> Create a new service.
- `docker service ls` —> List services in the swarm.
- `docker service ps <service-name>` —> List the tasks of a service.
- `docker service inspect <service-name>` —> Display detailed information about a service.
- `docker service logs <service-name>` —> Fetch logs from tasks in a service.
- `docker service scale <service-name>=<replicas>` —> Scale a service up or down.
- `docker service update [options] <service-name>` —> Update a service's configuration (image, args, etc.).
- `docker service rm <service-name>` —> Remove a service. 

<hr class="hr-light"/>

#### System & Info
- `docker info` —> Display system-wide information about the Docker installation.
- `docker version` —> Show the Docker version information.
- `docker system df` —> Show docker disk usage (images, containers, volumes).
- `docker system prune` —> Remove all unused data (containers, images, networks, build cache).
- `docker system prune -a` —> Remove all unused images, not just dangling ones.
- `docker image prune` —> Remove unused images.
- `docker container prune` —> Remove all stopped containers.
- `docker volume prune` —> Remove unused volumes.

---

#### Important Notes
- Adding -it in the run command allows you to run the container or image in interactive mode, where `-i` stands for interactive (keeps STDIN open so you can type commands), and `-t` allocates a pseudo-TTY (gives you a terminal interface with proper formatting), enabling you to interact directly with the container as if you're inside a shell.
- Adding -d in the run command runs the container in detached mode, meaning it runs in the background without attaching to your terminal. This allows the container to continue running independently of your session, and Docker will print the container ID so you can manage it later using commands like `docker logs`, `docker stop`, or `docker exec`. Detached mode is ideal for long-running services or daemons.
- You can configure the container's restart policy by adding `--restart` to the `docker run` command, followed by one of the available options: `always` (restarts the container if the main process exits or if the Docker daemon restarts), `unless-stopped`(restarts if the main process is killed but doesn't if Docker daemon restarts), - or `on-failure` (restarts only if the process exits with a non-zero status code or if the Docker daemon restarts). All policies don't restart the container if you manually stop it.
- Commands executed during the creation or setup of a container are called **build-time commands**, while those executed while the container is running are referred to as **runtime commands**.
- Docker allows you to pull official images directly, as they are published and maintained by the official organizations. To pull an unofficial image created by an individual developer, you must specify the account name before the image name, e.g., `docker image pull [account-name]/[image-name]`.