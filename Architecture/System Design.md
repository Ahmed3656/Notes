
## **Concepts**
- **Vertical Scaling (Scaling Up):** adding more resources (CPU, RAM, storage, etc.) to a single server to handle increased load.
	- Pros: easily applicable and doesn't require changes in the application architecture.
	- Cons: limited by hardware constraints and the costs rise exponentially at high-end configurations.
- **horizontal scaling (Scaling Out):** adding more servers/machines to distribute the workload using load balancers or reverse proxies.
	- Pros: can be scaled nearly infinitely and uses many less powerful, cheaper machines.
	- Cons: much more complex to implement and maintain. It also requires careful handling of data consistency, state, and fault tolerance.

<hr class="hr-light" />

- **Content Delivery Networks (CDNs):** a geographically distributed network of servers that cache and deliver static content (images, CSS, JS, videos) closer to end-users.
	- They improve latency and reduce server load.
	- They do not process application logic, they only serve cached assets.
	- Examples: Cloudflare, Akamai, Fastly

<hr class="hr-light" />

- **Sharding:** splitting a large database into smaller, independent shards stored on different servers. This improves performance, scalability, and query efficiency for massive datasets.
	**Challenges:**
    - Managing cross-shard queries.
    - Ensuring balanced data distribution.
    - Handling re-sharding when datasets grow unevenly.
- **Replication:** maintaining multiple synchronized copies of the same data across different servers.
	- **Types:**
		- **Master-Slave (Primary-Replica):** writes go to the master; replicas handle reads.
		- **Multi-Master:** multiple nodes accept writes; used in distributed databases.
	- Pros: high availability and fault tolerance, in addition to better read scalability.
	- Cons: potential consistency issues depending on replication strategy.
- **[CAP Theorem:](../Database/09.%20CAP%20Theorem.md)**
	In distributed systems, you can only guarantee **two out of three** properties:
	- **Consistency (C):** all nodes return the same data.
	- **Availability (A):** system always responds, even if some nodes fail.
	- **Partition Tolerance (P):** system continues working despite network partitions.
	**Implication:**
	- **CP** systems favor correctness over uptime (e.g., traditional databases).
	- **AP** systems favor uptime over strict correctness (e.g., DynamoDB, Cassandra).

<hr class="hr-light" />

#### **Caching**
Storing frequently accessed data in-memory or at the edge for faster retrieval.
- **Cache-Aside (Lazy Loading):** the application is responsible for loading data into cache when needed. Cache acts as a side storage that the application manages.
	- Characteristics: lazy loading approach / application control cache / cache miss triggers DB read / risk of "cache miss storms" on popular expired keys
	- Flow —> check cache for data  ──> cache hit —> return the data
		└──> cache miss —> query the database —> store the result in the cache —> return the data to the application
	- **Used for read-heavy applications with unpredictable access patterns.** The most common general-purpose pattern
- **Write-Through:** data is written to both cache and database simultaneously. Cache remains consistent with the database at all times.
	- Characteristics: synchronous writes / strong consistency / higher write latency / cache always up-to-date
	- Flow —> application writes data —> write to cache first —> write to database —> confirm both writes are successful —> return success to application
	- **Used for applications that require strong consistency and data durability (e.g., financial systems),** often combined with Cache-Aside for reads.
- **Write-Behind (Write-Back):** data is written to cache immediately, but database writes are deferred and happen asynchronously in the background.
	- Characteristics: asynchronous writes / lower write latency / eventual consistency / risk of data loss if cache fails / can batch database writes for efficiency
	- Flow —> application writes to cache —> return success immediately —> queue the database write —> background process writes to database —> database eventually consistent
	- **Used for write-heavy applications where eventual consistency is acceptable (e.g., clickstream tracking, activity logging).**
- **Write-Around:** writes bypass the cache and go directly to the database. The cache is only populated on a read request (via a cache-miss).
	- Characteristics: avoids populating cache with write-only data / reduces cache pollution / combines with Cache-Aside for reads / cache contains only read-accessed data
	- Flow —> application writes data —> write only to database —> on a subsequent read request —> cache miss occurs —> data is loaded from database into cache via Cache-Aside
	- **Used for write-heavy workloads where written data is unlikely to be re-read immediately (e.g., logging, telemetry data, bulk uploads).**
- **Refresh-Ahead:** cache is proactively refreshed before expiration by predicting which data will be needed and updating it in the background.
	- Characteristics: proactive cache refresh / predictive loading / minimal cache misses / complex implementation / risk of wasteful refresh if predictions are wrong
	- Flow —> monitor cache access patterns —> predicts needed data —> refreshes cache before data expires —> serves fresh data on request —> no cache miss experienced
	- **Used for predictable access patterns with with strict latency requirements (e.g., pre-loading a user's likely feed first thing in the morning).**

<hr class="hr-light" />

#### **Load Balancing**
Distributing incoming network traffic across multiple servers to ensure no single server is overwhelmed, improving scalability, availability, and fault tolerance.
- **Static Load Balancing:** uses predefined, fixed rules to distribute requests without considering the real-time state of servers.
	- **Round Robin:** it is the simplest approach, it just rotates the requests evenly among the servers in sequence. **Used for homogeneous server clusters with evenly distributed requests.**
	- **Sticky (Session-Aware) Round Robin:** an extension of Round Robin that tries to send subsequent requests from the same user to the same server. The goal is to improve performance by keeping related data on the same server, so it maintains a **stateful** connection between the client and the server. Uneven load can occur as newly arriving users are assigned randomly, and it can also be problematic if a server goes down since those users lose their session context. **Used when session affinity is required (e.g., apps without centralized session storage).**
	- **Weighted Round Robin:** allows admins to assign different weights or priorities to different servers, where servers with higher weights will receive a proportionally higher number of requests. The downside is that the weights have to be manually configured, which is less adaptive to real-time changes. **Used when servers have different hardware capacity or performance profiles.**
	- **Random Selection:** each request is assigned to a random server. Simple, low-overhead, and avoids predictable patterns, but it doesn’t guarantee even distribution. **Used when traffic is light or evenly distributed by chance.**
	- **IP/URL Hash:** use a hash function to map incoming requests to the backend servers. The hash function often uses the client's IP address or the requested URL as input for determining where to route each request. It can evenly distribute requests if the function is chosen wisely, however, selecting an optimal hash function could be challenging. **Used for cache-heavy systems and when client affinity is required (e.g., CDNs, gaming servers).**
	- **Consistent Hashing:** a specialized hashing approach that, unlike simple IP/URL hashing, minimizes disruption when servers are added or removed, only a small portion of requests need to be remapped. **Used in distributed caches and databases (e.g., Cassandra, Memcached, CDN routing).**
	- **Geolocation-Based Routing:** routes users to the server closest to their geographical location to reduce latency. **Used for global applications (CDNs, DNS load balancing).**
- **Dynamic Load Balancing:** adapts routing decisions based on the current state of servers (connections, response time, resource usage).
	- **Least Connections:** sends each new request to the server with the least number of active connections. This adapts well when requests vary in processing time, but it assumes all connections cost about the same (which isn’t always true). **Used for applications with varying session lengths (e.g., chat apps, APIs with uneven request processing times).**
	-  **Weighted Least Connections:** similar to Least Connections but also considers server capacity (e.g., a server with twice the capacity can handle twice the connections). **Used for heterogeneous server clusters where hardware differs significantly.**
	- **Least Response Time:** sends incoming requests to the server with the lowest current latency or fastest response time. Latency for each server is continuously measured and factored in. This approach is highly adaptive and reactive, however, it requires constant monitoring which incurs significant overhead and introduces complexity. It also doesn't consider how many existing requests each server already has. **Used for latency-sensitive systems where fast response is critical (e.g., trading platforms, interactive apps).**
	- **Resource-Based (Load-Aware Scheduling):** requests are routed based on real-time metrics like CPU, memory, or I/O usage of servers. Requires monitoring agents on servers. **Used when workloads are heavy and uneven, ensuring more balanced resource consumption.**
	- **Least Bandwidth / Least Traffic:** routes to the server currently serving the least amount of data (measured in Mbps or throughput). **Used for streaming, file transfer, or content-heavy systems.**
	- **Response-Time + Connections Hybrid:** considers both response time and number of connections together (e.g., F5 load balancers). **Used for balancing latency-sensitive systems with uneven load.**
- **Health Checks:** a fundamental mechanism where the load balancer proactively monitors the health and availability of servers to prevent routing traffic to failed or unhealthy nodes. This is a critical feature for **fault tolerance**.
	- **Passive Health Checks (In-band):** the load balancer observes the real-time behavior of server responses (e.g., timeouts, connection failures, HTTP 5xx status codes) to infer their health status. A server generating too many errors is temporarily quarantined.
	- **Active Health Checks (Out-of-band):** the load balancer periodically sends separate health probe requests (e.g., a TCP SYN packet, an HTTP GET /health) to each server at a configurable interval. A server that fails to respond correctly is marked **down** and removed from the pool until it passes subsequent checks.

<hr class="hr-light" />

## **Other Key Concepts**
- **TCP/IP:** foundation of all internet communication, it is a reliable, ordered, connection-oriented protocol.
- **DNS:** resolves human-readable domain names into IP addresses.
- **HTTP/HTTPS:** web communication protocols; HTTPS adds encryption via TLS.
- **REST:** API style using HTTP methods and stateless resources.
- **GraphQL:** flexible query-based API allowing clients to request exactly the data they need.
- **gRPC:** high-performance RPC framework built on HTTP/2, ideal for microservices.
- **WebSockets:** enables persistent, bidirectional client-server communication in real time.
- **SQL vs. NoSQL:**
    - **SQL:** relational, structured schema, ACID compliance
    - **NoSQL:** non-relational, highly scalable, supports unstructured or semi-structured data
- **ACID Properties:** Atomicity, Consistency, Isolation, Durability the core principles for reliable database transactions.
- **Message Queues:** systems like **Kafka**, **RabbitMQ**, or **SQS** for asynchronous communication between services, improving scalability and decoupling components.

---

## **Important Notes**
- If the service is stateless it should be easy to scale horizontally.