
#### **Concepts**
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
- **CAP Theorem:**
	In distributed systems, you can only guarantee **two out of three** properties:
	- **Consistency (C):** all nodes return the same data.
	- **Availability (A):** system always responds, even if some nodes fail.
	- **Partition Tolerance (P):** system continues working despite network partitions.
	**Implication:**
	- **CP** systems favor correctness over uptime (e.g., traditional databases).
	- **AP** systems favor uptime over strict correctness (e.g., DynamoDB, Cassandra).

<hr class="hr-light" />

- **Caching:** storing frequently accessed data in-memory or at the edge for faster retrieval.
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

**Other Key Concepts**
- **TCP/IP:** foundation of all internet communication — reliable, ordered, connection-oriented protocol.
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

#### **Important Notes**
- If the service is stateless it should be easy to scale horizontally.