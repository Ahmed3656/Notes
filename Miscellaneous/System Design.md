
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

**Other Key Concepts**
- **Caching:** Storing frequently accessed data in-memory or at the edge for faster retrieval.
- **TCP/IP:** Foundation of all internet communication — reliable, ordered, connection-oriented protocol.
- **DNS:** Resolves human-readable domain names into IP addresses.
- **HTTP/HTTPS:** Web communication protocols; HTTPS adds encryption via TLS.
- **REST:** API style using HTTP methods and stateless resources.
- **GraphQL:** Flexible query-based API allowing clients to request exactly the data they need.
- **gRPC:** High-performance RPC framework built on HTTP/2, ideal for microservices.
- **WebSockets:** Enables persistent, bidirectional client-server communication in real time.
- **SQL vs. NoSQL:**
    - **SQL:** Relational, structured schema, ACID compliance
    - **NoSQL:** Non-relational, highly scalable, supports unstructured or semi-structured data
- **ACID Properties:** Atomicity, Consistency, Isolation, Durability — core principles for reliable database transactions.
- **Message Queues:** Systems like **Kafka**, **RabbitMQ**, or **SQS** for asynchronous communication between services, improving scalability and decoupling components.

---

#### **Important Notes**
- If the service is stateless it should be easy to scale horizontally.