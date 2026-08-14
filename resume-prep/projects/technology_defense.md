# Technology Defense Master File

This document provides a 7-level defense for every significant technology on the resume.

## 1. Redis

### Level 1 — Basic
**Q: What is Redis?**
**A:** Redis is an in-memory data structure store, used as a distributed, in-memory key-value database, cache, and message broker.

#### Follow-up: Why use it over a Python dictionary?
**A:** A Python dictionary is local to a single process. If I have 4 FastAPI workers, they don't share memory. Redis provides a centralized, distributed state that all workers can access with microsecond latency.

#### Follow-up: What data structures does it support?
**A:** Strings, Hashes, Lists, Sets, Sorted Sets, Bitmaps, HyperLogLogs, and Streams.

### Level 2 — Project-Specific
**Q: Why did you use Redis in your project?**
**A:** > **Needs candidate-specific confirmation** I used Redis in LLM Cost Guard for distributed rate-limiting using the token bucket algorithm, and in RAGOS for caching API usage metrics.

#### Follow-up: Why not do rate limiting in Postgres?
**A:** Postgres hits disk and has transactional overhead. For rate limiting, I need sub-millisecond checks before routing requests. Redis does this entirely in RAM.

#### Follow-up: How exactly did you structure the keys?
**A:** I used a string key like `ratelimit:{user_id}:{endpoint}` and used `INCR` along with `EXPIRE` to maintain rolling counts.

### Level 3 — Mechanism
**Q: How does Redis work internally?**
**A:** Redis is single-threaded for command execution. It uses an event loop (epoll) to multiplex connections. Because it's single-threaded, commands are atomic by default.

#### Follow-up: If it's single-threaded, how does it handle thousands of connections?
**A:** I/O multiplexing. The kernel notifies Redis when a socket is readable. Redis reads the command, executes it in memory instantly, and writes the response back. The execution takes nanoseconds.

#### Follow-up: What about long commands blocking the thread?
**A:** That's the main danger. A command like `KEYS *` scans the entire keyspace and blocks everything. We use `SCAN` instead, which returns a cursor and a small batch.

### Level 4 — Tradeoffs
**Q: Why Redis instead of Memcached?**
**A:** Memcached only supports strings and lacks persistence. I needed advanced structures (like Sorted Sets for leaderboards) and optional persistence (AOF/RDB).

#### Follow-up: When would you actually choose Memcached?
**A:** Only if I need simple string caching and want to leverage multi-threading out-of-the-box for sheer throughput on a single massive machine.

### Level 5 — Failure
**Q: What happens if Redis goes down?**
**A:** If used as a cache (Cache-Aside), the app falls back to the database, causing a Cache Stampede.

#### Follow-up: How do you prevent a Cache Stampede?
**A:** We can use probabilistic early expiration or a distributed lock. If a key is expired, the first thread gets a lock, queries the DB, and repopulates the cache, while others wait or return stale data.

### Level 6 — Debugging
**Q: Redis memory is full. How do you debug?**
**A:** I check `INFO memory` to see `used_memory`. Then I check the `maxmemory-policy`.

#### Follow-up: What eviction policy do you use?
**A:** `volatile-lru`. It evicts the least recently used keys, but only among those that have an expiration set. This protects permanent keys.

#### Follow-up: How do you find what's taking up space?
**A:** I run `redis-cli --bigkeys` to safely sample the database for large keys without blocking the main thread.

### Level 7 — Scaling
**Q: How do you scale Redis?**
**A:** For read scaling, I use asynchronous Replication (master-slave).

#### Follow-up: What if the master dies?
**A:** I use Redis Sentinel. It monitors the nodes and automatically promotes a replica to master if the primary fails.

#### Follow-up: What if data exceeds one machine's RAM?
**A:** I use Redis Cluster. It shards data across multiple masters using hash slots. Keys are hashed using CRC16 modulo 16384 to determine their node.

---

## 2. Docker

### Level 1 — Basic
**Q: What is Docker?**
**A:** Docker is a platform for containerizing applications, packaging code with its dependencies into isolated units.

#### Follow-up: How is a container different from a VM?
**A:** VMs virtualize the hardware and run a full Guest OS. Containers virtualize the OS, sharing the host's kernel. This makes containers start in milliseconds and use megabytes of RAM instead of gigabytes.

### Level 2 — Project-Specific
**Q: Why did you use Docker?**
**A:** > **Needs candidate-specific confirmation** To ensure consistency between my Mac development environment and the Linux production server for the FastAPI backend and workers.

#### Follow-up: Did you use docker-compose?
**A:** Yes, for local development, I used `docker-compose` to spin up FastAPI, Postgres, and Redis simultaneously with a single command.

### Level 3 — Mechanism
**Q: How does Docker work under the hood?**
**A:** It uses Linux Namespaces for isolation (process IDs, network, mounts) and Control Groups (cgroups) for resource limitation (CPU, memory).

#### Follow-up: How does the file system work?
**A:** It uses a Union File System (OverlayFS). Images are built in read-only layers. When a container runs, a thin read-write layer is added on top.

### Level 4 — Tradeoffs
**Q: What are the disadvantages of Docker?**
**A:** Weaker security isolation than VMs. If there's a kernel vulnerability, a container breakout can compromise the host and all other containers.

### Level 5 — Failure
**Q: What happens if a container runs out of memory?**
**A:** The host OS kernel's OOM (Out of Memory) Killer terminates the container process.

#### Follow-up: How do you prevent one container from crashing the host?
**A:** I set memory limits in `docker-compose.yml` or `docker run` using `--memory`.

### Level 6 — Debugging
**Q: A container starts and immediately exits. How do you debug?**
**A:** I run `docker logs <container_id>`. If it's empty, I run `docker inspect` to check the `ExitCode`.

#### Follow-up: What if you need to look at the files inside?
**A:** I override the entrypoint: `docker run -it --entrypoint /bin/sh <image_name>` to poke around the filesystem before the app crashes.

### Level 7 — Scaling
**Q: How do you handle multiple Docker containers in production?**
**A:** I wouldn't manage them manually. I'd use Kubernetes to handle scheduling, networking, and autoscaling across a cluster of nodes.

---

## 3. PostgreSQL

### Level 1 — Basic
**Q: What is PostgreSQL?**
**A:** It is a powerful, open-source object-relational database system known for strict ACID compliance and advanced features.

#### Follow-up: What does ACID mean?
**A:** Atomicity (all or nothing), Consistency (valid states), Isolation (concurrent transactions don't interfere), and Durability (committed data is saved).

### Level 2 — Project-Specific
**Q: Why use Postgres in your project?**
**A:** > **Needs candidate-specific confirmation** For LLM Cost Guard, billing data requires absolute accuracy and relational integrity. I also used `pgvector` for embedding storage, avoiding a separate vector DB.

### Level 3 — Mechanism
**Q: How does Postgres handle concurrent transactions?**
**A:** It uses MVCC (Multi-Version Concurrency Control). When you update a row, Postgres doesn't overwrite it; it writes a new version. Readers read the old version, writers write the new one. They don't block each other.

#### Follow-up: Doesn't that waste disk space?
**A:** Yes, it creates "dead tuples". The `VACUUM` process runs in the background to reclaim that space.

### Level 4 — Tradeoffs
**Q: Why Postgres instead of MongoDB?**
**A:** Postgres enforces strict schemas which prevents bad data from entering the system. With JSONB, Postgres can handle document storage almost as well as Mongo, while still allowing relational joins.

### Level 5 — Failure
**Q: What happens if Postgres crashes mid-transaction?**
**A:** When it restarts, it reads the Write-Ahead Log (WAL). Any committed transaction in the WAL is replayed. Uncommitted transactions are rolled back.

### Level 6 — Debugging
**Q: A query is suddenly very slow. How do you debug?**
**A:** I prefix the query with `EXPLAIN ANALYZE`. This executes the query and shows the query plan and actual execution times.

#### Follow-up: What are you looking for in the output?
**A:** I look for `Seq Scan` (Sequential Scan) on large tables, which means it's reading every row. I'd add an index to turn it into an `Index Scan`.

### Level 7 — Scaling
**Q: How do you scale Postgres?**
**A:** I start with a connection pooler like PgBouncer because Postgres processes are heavy. Then I add read replicas using Streaming Replication.

#### Follow-up: How do you scale writes?
**A:** Write scaling is hard. I'd look into Table Partitioning (e.g., partitioning logs by month). If that fails, application-level sharding.

---

## 4. FastAPI

### Level 1 — Basic
**Q: What is FastAPI?**
**A:** A modern, high-performance web framework for Python APIs based on standard type hints.

#### Follow-up: What makes it fast?
**A:** It is built on Starlette for ASGI networking and Pydantic for fast data validation. It inherently supports asynchronous programming.

### Level 2 — Project-Specific
**Q: Where did you use FastAPI?**
**A:** > **Needs candidate-specific confirmation** It was the main API gateway in RAGOS, handling incoming query requests and orchestrating DB and LLM calls.

### Level 3 — Mechanism
**Q: How does validation work?**
**A:** Through dependency injection and Pydantic. When a request arrives, Pydantic parses the JSON payload according to the Python type hints on the model, throwing a 422 error automatically if invalid.

### Level 4 — Tradeoffs
**Q: Why not use Django?**
**A:** Django is monolithic and includes templates, ORMs, and admin panels. For a microservice API that just returns JSON, FastAPI is lighter, faster, and natively async.

### Level 5 — Failure
**Q: What happens if a FastAPI request throws an unhandled exception?**
**A:** The ASGI server catches it, returns a 500 Internal Server Error, and continues serving other requests. The worker doesn't crash unless it's a critical memory error.

### Level 6 — Debugging
**Q: Your API is completely locked up. No requests are completing. Why?**
**A:** I probably put a synchronous, blocking call (like `requests.get()` or a heavy CPU computation) inside an `async def` endpoint. This blocks the single event loop thread.

#### Follow-up: How do you fix it?
**A:** Either change it to a `def` endpoint (so FastAPI runs it in a background threadpool) or use an async library like `httpx` instead of `requests`.

### Level 7 — Scaling
**Q: How do you scale FastAPI?**
**A:** Run it with a process manager like Gunicorn using Uvicorn workers (`-k uvicorn.workers.UvicornWorker`). Then deploy multiple containers behind a load balancer.

---

## 5. Kubernetes (K8s)

### Level 1 — Basic
**Q: What is Kubernetes?**
**A:** A container orchestration platform that automates deployment, scaling, and operations of application containers.

### Level 2 — Project-Specific
**Q: How did you use Kubernetes?**
**A:** > **Needs candidate-specific confirmation** I used it to deploy the microservices (API, workers). I wrote YAML manifests for Deployments, Services, and Ingress to manage the application lifecycle.

### Level 3 — Mechanism
**Q: How does Kubernetes know to restart a container?**
**A:** I configure Liveness Probes. The Kubelet on the node periodically pings a health endpoint (e.g., `/health`). If it fails, Kubelet restarts the container.

#### Follow-up: What about Readiness Probes?
**A:** Readiness probes check if the app is ready to receive traffic. If it fails, K8s removes the Pod from the Service load balancer, but doesn't restart it.

### Level 4 — Tradeoffs
**Q: Why use K8s instead of Docker Compose?**
**A:** Docker Compose is for single nodes. K8s handles multi-node clusters, self-healing, rolling updates, and autoscaling, though it adds massive complexity.

### Level 5 — Failure
**Q: What happens if a Node dies?**
**A:** The control plane notices the node is unresponsive. The ReplicaSet controller sees that the desired number of Pods is not met, and schedules those Pods onto healthy nodes.

### Level 6 — Debugging
**Q: A Pod is in ImagePullBackOff. What do you do?**
**A:** I check `kubectl describe pod`. It usually means the image tag doesn't exist, there's a typo in the registry URL, or the cluster lacks authentication secrets (ImagePullSecrets) for a private registry.

### Level 7 — Scaling
**Q: How do you autoscale in K8s?**
**A:** I use Horizontal Pod Autoscaler (HPA). I set a target CPU utilization (e.g., 80%). If usage spikes, HPA increases the `replicas` in the Deployment automatically.

---

## 6. Celery & RabbitMQ

### Level 1 — Basic
**Q: What are Celery and RabbitMQ?**
**A:** Celery is an asynchronous distributed task queue. RabbitMQ is the message broker it uses to send and receive messages.

### Level 2 — Project-Specific
**Q: Why use a task queue?**
**A:** > **Needs candidate-specific confirmation** In RAGOS, embedding generation and LLM calls take seconds. I can't block the FastAPI HTTP response for that long. I push a task to Celery, return a 202 status, and process the LLM call in the background.

### Level 3 — Mechanism
**Q: How does a task get executed?**
**A:** FastAPI serializes the task arguments and sends them to a RabbitMQ queue. A Celery worker, polling that queue, pops the message, executes the function, and stores the result in a backend (like Redis).

### Level 4 — Tradeoffs
**Q: Why RabbitMQ instead of Redis as a broker?**
**A:** Redis is an in-memory datastore acting as a broker; RabbitMQ is a dedicated message broker built for reliability, complex routing (exchanges/bindings), and persistent queues.

### Level 5 — Failure
**Q: What happens if a worker dies mid-task?**
**A:** If configured with `acks_late=True`, RabbitMQ won't receive the acknowledgment. It will re-queue the message, and another worker will process it. Tasks must be idempotent to handle this safely.

### Level 6 — Debugging
**Q: Tasks are stuck in the queue. How do you debug?**
**A:** I check the RabbitMQ management UI. If there are messages but no consumers, the workers are down. If there are consumers, they might be deadlocked on a bad network request without a timeout.

### Level 7 — Scaling
**Q: How do you scale workers?**
**A:** Spin up more Celery worker containers. For I/O-bound tasks, increase the concurrency of existing workers using threads or gevent pools.

---

## 7. Python

### Level 1 — Basic
**Q: How is memory managed in Python?**
**A:** Python uses Reference Counting as its primary memory management, backed by a Garbage Collector to detect reference cycles.

#### Follow-up: What is a reference cycle?
**A:** When object A references object B, and object B references object A. Their reference counts will never drop to zero, so the cycle garbage collector is needed to clean them up.

### Level 3 — Mechanism
**Q: What is the GIL?**
**A:** The Global Interpreter Lock. It is a mutex that protects access to Python objects, preventing multiple native threads from executing Python bytecodes at once.

#### Follow-up: How do you bypass the GIL?
**A:** By using the `multiprocessing` module (which spawns entirely separate OS processes) or writing C-extensions that release the GIL during heavy computation.

### Level 4 — Tradeoffs
**Q: List Comprehension vs Generator Expression?**
**A:** `[x * 2 for x in huge_list]` computes everything and stores it in RAM. `(x * 2 for x in huge_list)` creates a generator object that yields one item at a time, saving memory at the cost of slight CPU overhead.

### Level 6 — Debugging
**Q: You have a memory leak in a Python app. How do you find it?**
**A:** I use the `tracemalloc` module to trace memory allocations, take snapshots at different times, and compare them to see which lines of code are allocating memory that isn't being freed.

---

## 8. AsyncIO

### Level 1 — Basic
**Q: What is AsyncIO?**
**A:** It's a library for writing concurrent code using the `async`/`await` syntax.

### Level 3 — Mechanism
**Q: How does the event loop work?**
**A:** It runs in a single thread. It executes coroutines until they hit an `await` on an I/O operation. Control is yielded back to the loop, which uses `epoll`/`select` to monitor I/O. When the I/O is ready, the loop resumes the coroutine.

### Level 4 — Tradeoffs
**Q: AsyncIO vs Multi-threading?**
**A:** Threads incur context-switching overhead by the OS. AsyncIO does cooperative multitasking in user-space, which is much lighter, allowing thousands of concurrent connections on a single core.

### Level 6 — Debugging
**Q: How do you catch blocking code in AsyncIO?**
**A:** Run Python with `PYTHONASYNCIODEBUG=1`. The event loop logs warnings when a coroutine blocks the loop for more than 100 milliseconds.

---

## 9. Git

### Level 1 — Basic
**Q: What is a Git commit conceptually?**
**A:** A commit is a snapshot of the repository at a specific point in time, identified by a SHA-1 hash.

### Level 3 — Mechanism
**Q: Rebase vs Merge?**
**A:** `Merge` creates a new merge commit, preserving the exact history. `Rebase` rewrites history by moving your feature branch's base to the tip of main, creating a linear history.

### Level 6 — Debugging
**Q: You messed up a rebase and lost code. How do you recover?**
**A:** I use `git reflog`. It tracks every time the tip of branches are updated locally. I can find the hash before the rebase and `git reset --hard` to it.

---

## 10. YOLOv8, OpenCV, MediaPipe

### Level 1 — Basic
**Q: What is YOLO?**
**A:** You Only Look Once is a real-time object detection system. YOLOv8 is an anchor-free architecture, making it faster and more accurate.

### Level 2 — Project-Specific
**Q: How did you use them in Focus Lock?**
**A:** > **Needs candidate-specific confirmation** I used OpenCV to capture frames, MediaPipe for fast facial landmark detection (mesh), and YOLOv8 for more robust bounding box detection when needed.

### Level 3 — Mechanism
**Q: What is Non-Maximum Suppression (NMS)?**
**A:** YOLO often predicts multiple bounding boxes for the same object. NMS removes redundant boxes by keeping the one with the highest confidence and suppressing others that have a high Intersection over Union (IoU) with it.

---

## 11. Vector Databases (ChromaDB & pgvector)

### Level 1 — Basic
**Q: What is a Vector Database?**
**A:** A database optimized for storing and querying high-dimensional vectors (embeddings) using similarity metrics like Cosine Similarity.

### Level 3 — Mechanism
**Q: How do they search so fast?**
**A:** They don't do exact nearest neighbor searches (which require scanning everything). They use Approximate Nearest Neighbor (ANN) algorithms, like HNSW (Hierarchical Navigable Small World), which uses graph layers to find neighbors quickly.

### Level 4 — Tradeoffs
**Q: ChromaDB vs pgvector?**
**A:** ChromaDB is a dedicated vector store, great for rapid prototyping. `pgvector` is an extension for PostgreSQL, allowing you to store vectors alongside relational data, ensuring ACID compliance and avoiding infrastructure sprawl.

---

## 12. System Design & Architecture Concepts

### Level 1 — Distributed Systems
**Q: What is the CAP Theorem?**
**A:** A distributed system can only guarantee two out of three: Consistency (all nodes see the same data), Availability (every request receives a response), and Partition tolerance (system operates despite network drops). Because partitions (P) always happen, you must choose between CP and AP.

#### Follow-up: How does Postgres fit in?
**A:** A standard Postgres primary-replica setup prioritizes Consistency. If the primary cannot sync with replicas synchronously, it might block writes (sacrificing Availability) to remain Consistent.

### Level 2 — Caching Strategies
**Q: What are the main caching patterns?**
**A:**
1. **Cache-Aside:** Application checks cache. If miss, queries DB, updates cache. (Most common, used in my Redis setups).
2. **Write-Through:** Application writes to cache, which synchronously writes to DB.
3. **Write-Behind:** Application writes to cache, which asynchronously writes to DB.

### Level 3 — Event-Driven Architecture
**Q: What is Event-Driven Architecture (EDA)?**
**A:** Services communicate by emitting and reacting to events via a broker (RabbitMQ/Kafka) rather than synchronous API calls.

#### Follow-up: What is the main challenge with EDA?
**A:** Eventual consistency and debugging. If a transaction spans 4 services, tracking failures requires distributed tracing (like OpenTelemetry) and compensating transactions (Sagas) to rollback.

### Level 4 — API Design
**Q: What makes an API RESTful?**
**A:** It is stateless, cacheable, uses standard HTTP methods (GET, POST, PUT, DELETE), uses standard status codes, and exposes resources via URIs.

#### Follow-up: What is idempotency?
**A:** An operation is idempotent if doing it once has the same effect as doing it multiple times. PUT and DELETE should be idempotent. POST is usually not.

---

## 13. AWS (Amazon Web Services)

### Level 1 — Basic
**Q: What AWS services are you familiar with?**
**A:** > **Needs candidate-specific confirmation** EC2 for virtual servers, S3 for object storage, RDS for managed databases, and IAM for identity/security.

### Level 3 — Mechanism
**Q: What is an S3 bucket?**
**A:** It is object storage, not block storage. You store files with metadata under a key. It's highly durable (99.999999999%) and replicated across Availability Zones.

---

## 14. Linux & OpenTelemetry

### Level 1 — Basic (Linux)
**Q: How do you find a process using a specific port?**
**A:** `sudo lsof -i :8080` or `netstat -tulpn | grep 8080`.

**Q: How do you check disk space?**
**A:** `df -h` for mounted filesystems, `du -sh *` to see sizes of directories.

### Level 1 — Basic (OpenTelemetry)
**Q: What are the three pillars of observability?**
**A:** Logs (discrete events), Metrics (aggregations over time, like CPU %), and Traces (request lifecycle across microservices).

#### Follow-up: How do traces work?
**A:** A Trace ID is generated at the API gateway and passed in HTTP headers to every downstream microservice, allowing you to reconstruct the entire request path.

---

## 15. GitHub Actions

### Level 1 — Basic
**Q: What is a GitHub Action?**
**A:** It is a CI/CD platform integrated into GitHub. You define workflows in YAML to automatically build, test, and deploy code on specific triggers (like `push` or `pull_request`).

### Level 3 — Mechanism
**Q: How do runners work?**
**A:** GitHub provides hosted virtual machines (Runners) that execute the jobs in isolated environments. You can also host your own runners for security or cost reasons.

---

## 16. C++ / C / JavaScript

*(Defensive coverage - Acknowledge specific usage rather than deep expertise if not primary)*

**Q: Where did you use C++?**
**A:** > **Needs candidate-specific confirmation** Primarily for hardware/firmware integration (like ArduPilot), focusing on memory management via pointers and basic standard template library (STL) usage.

**Q: Where did you use JavaScript?**
**A:** > **Needs candidate-specific confirmation** For basic frontend interactions or scripting, understanding the V8 event loop and async/await, though my primary backend focus is Python.

---
**End of Document**

---

## Deep Dive Supplement

### 1. Redis - Deeper Follow-ups

#### Caching Strategies Deep Dive
**Q: How did you decide between Cache-Aside and Write-Through?**
**A:** In RAGOS, embedding generation is computationally expensive. Writing to the cache on every read miss (Cache-Aside) is appropriate because read patterns (evaluating similar prompts) are highly repetitive. A write-through approach would mean preemptively generating embeddings for data that might never be queried.

#### Follow-up: What if the database data changes out-of-band?
**A:** That's the main danger of Cache-Aside. We use TTLs (Time To Live) as a basic defense mechanism. If we need strict consistency, we implement Cache Invalidation on write, sending a message to Redis to `DEL` the key when the underlying Postgres record is updated.

#### Persistence Deep Dive
**Q: What's the difference between RDB and AOF?**
**A:** RDB takes point-in-time snapshots of the dataset at specified intervals. AOF (Append Only File) logs every write operation received by the server. RDB is faster to restart from but can lose minutes of data. AOF is more durable (fsync every second) but results in a larger file and slower restarts.

#### Follow-up: Which one did you use?
**A:** > **Needs candidate-specific confirmation** For rate-limiting in Cost Guard, data loss on restart is acceptable (users get a fresh quota), so RDB is sufficient. For critical state, we would enable both.

#### Lua Scripting Deep Dive
**Q: Why use Lua scripts in Redis?**
**A:** To perform complex, atomic operations. For example, a token bucket rate limiter requires getting the current token count, calculating elapsed time, refilling the bucket, checking if tokens are sufficient, and decrementing them. If done via multiple client calls, race conditions occur. A Lua script executes entirely on the Redis server atomically.

### 2. Docker - Deeper Follow-ups

#### Dockerfile Anatomy
**Q: Walk me through an optimized Dockerfile.**
**A:** Start with a lightweight base image like `python:3.11-slim`. Set a non-root user for security. Use multi-stage builds. First stage: install build dependencies (gcc, python3-dev), compile wheels. Second stage: copy only the compiled wheels and app code. This reduces image size dramatically and minimizes vulnerabilities.

#### Follow-up: How does layer caching work?
**A:** Each instruction in a Dockerfile creates a layer. If the instruction and files haven't changed, Docker reuses the cached layer. Therefore, we always copy `requirements.txt` and run `pip install` BEFORE copying the rest of the application code, so code changes don't invalidate the expensive `pip install` cache layer.

#### Networking Deep Dive
**Q: What is the difference between bridge and host networking?**
**A:** Bridge creates a private internal network for containers on the same host, requiring port mapping (`-p 8080:80`) to reach the outside. Host networking (`--network host`) bypasses this isolation, binding the container directly to the host's network interfaces, improving performance but risking port collisions.

### 3. PostgreSQL - Deeper Follow-ups

#### Indexing Deep Dive
**Q: When would you use a B-tree index vs a GIN index?**
**A:** B-trees are the default and perfect for equality (`=`) and range queries (`>`, `<`) on scalar data like integers or timestamps. GIN (Generalized Inverted Index) is designed for composite values like arrays or JSONB, allowing fast searches inside the documents, like finding all rows where a JSONB column contains a specific key-value pair.

#### Follow-up: What about GiST?
**A:** GiST is useful for spatial data (PostGIS) or nearest-neighbor searches, which is highly relevant for vector search (`pgvector`) before HNSW was fully adopted.

#### Connection Pooling Deep Dive
**Q: Why is PgBouncer necessary?**
**A:** Postgres spawns a new OS process (using ~10MB RAM) for every connection. If 1000 FastAPI workers open a connection, Postgres spends all its time context-switching and thrashing memory. PgBouncer sits in front, maintaining a small pool of actual DB connections (e.g., 50) and multiplexing the 1000 client connections onto them.

#### Follow-up: Transaction vs Session pooling?
**A:** Transaction pooling is best for high concurrency. A client gets a DB connection only for the duration of a single transaction, then returns it. Session pooling keeps the connection for the entire life of the client session, which is less efficient for APIs.

### 4. FastAPI - Deeper Follow-ups

#### ASGI vs WSGI Deep Dive
**Q: What is the difference between ASGI and WSGI?**
**A:** WSGI (used by Django/Flask) is synchronous. It handles one request per thread. ASGI is asynchronous. It uses an event loop, allowing a single thread to handle thousands of concurrent requests by yielding control during I/O waits (like DB queries).

#### Dependency Injection Deep Dive
**Q: How do you use Dependency Injection in FastAPI?**
**A:** I use `Depends()`. For example, a dependency `get_db()` yields a database session. I inject this into my route handlers. This isolates the DB connection logic, makes unit testing incredibly easy (by overriding the dependency with a mock), and ensures connections are properly closed after the request.

#### Follow-up: How do you handle authentication?
**A:** I create an `oauth2_scheme` dependency that extracts the JWT token from the Authorization header. My endpoint declares `user = Depends(get_current_user)`, which in turn depends on the token extractor, decodes the token, and returns the User object, throwing a 401 if invalid.

### 5. Kubernetes - Deeper Follow-ups

#### Deployment Strategy Deep Dive
**Q: How do you update a live application in Kubernetes without downtime?**
**A:** I use a Rolling Update strategy on the Deployment. I configure `maxSurge` (how many extra pods can be created during rollout) and `maxUnavailable` (how many pods can be taken down). K8s spins up new pods, waits for their Readiness probes to pass, then routes traffic to them while terminating old pods.

#### Follow-up: What if the new code has a bug?
**A:** If liveness/readiness probes fail, the rollout pauses. I can run `kubectl rollout undo deployment/<name>` to instantly revert to the previous ReplicaSet.

#### ConfigMaps and Secrets Deep Dive
**Q: How do you pass environment variables to your Pods?**
**A:** Non-sensitive data (like API URLs) goes in ConfigMaps. Sensitive data (like DB passwords or OpenAI API keys) goes in Secrets. These are injected into the Pods as environment variables or mounted as read-only files. Note that base Secrets are just base64 encoded, not encrypted by default, so I'd use something like AWS KMS or HashiCorp Vault for real production security.

### 6. Celery & RabbitMQ - Deeper Follow-ups

#### Dead Letter Queues
**Q: What happens to a task that fails repeatedly?**
**A:** We configure a Dead Letter Exchange (DLX) in RabbitMQ. If a message is rejected (e.g., parsing error) or its TTL expires, it gets routed to the DLX, which puts it in a Dead Letter Queue. We can then inspect this queue manually to debug the poison messages without them infinitely crashing workers.

#### Task Idempotency
**Q: Why is idempotency critical for Celery tasks?**
**A:** Because message brokers guarantee "at least once" delivery. If a network blip occurs after a task finishes but before the worker sends the ACK to RabbitMQ, RabbitMQ will deliver the task again. If the task is charging a credit card, the user gets double-charged. Idempotency means checking a database state (e.g., `if payment_processed: return`) before acting.

### 7. Python - Deeper Follow-ups

#### Context Managers Deep Dive
**Q: What is a context manager?**
**A:** It's a protocol implemented using `__enter__` and `__exit__` methods, used with the `with` statement. It ensures resources (like file handles or DB connections) are cleanly initialized and reliably torn down, even if exceptions occur.

#### Follow-up: How would you write one without a class?
**A:** I can use the `@contextmanager` decorator from `contextlib`. I write a generator function, yield the resource, and put the cleanup code in a `finally` block after the yield.

#### Decorators Deep Dive
**Q: What is a decorator and how do you write one?**
**A:** It's a function that takes another function, extends its behavior without modifying its core, and returns a new function. It's heavily used in FastAPI (e.g., `@app.get()`). To write one, I define an outer function that receives the target function, and an inner wrapper function (using `*args, **kwargs`) that adds the behavior before/after calling the target.

### 8. AsyncIO - Deeper Follow-ups

#### Task Gathering Deep Dive
**Q: How do you make 10 API calls concurrently?**
**A:** I wouldn't await them one by one in a loop (which makes them sequential). I would create a list of coroutines, and then use `await asyncio.gather(*coroutines)`. This schedules them all on the event loop concurrently and returns a list of results when all are done.

#### Run in Executor
**Q: What if you absolutely must run a synchronous, CPU-bound library inside an async FastAPI route?**
**A:** I use `loop.run_in_executor(None, sync_func)`. This offloads the blocking execution to a separate thread pool (or process pool), preventing it from freezing the main asyncio event loop.

### 9. Git - Deeper Follow-ups

#### Cherry-Pick Deep Dive
**Q: What is cherry-picking?**
**A:** It's applying the changes introduced by one specific existing commit onto the current branch. It's useful for backporting a bug fix from `main` to an older release branch without merging all the other new features.

#### Bisect Deep Dive
**Q: Someone broke the build and you don't know which commit did it. How do you find it?**
**A:** `git bisect`. It uses binary search. I mark the current broken state as `bad`, and an older known working commit as `good`. Git automatically checks out a commit in the middle. I test it. If it works, I tell git `good`, if it fails, I tell git `bad`. It halves the search space each time, finding the exact broken commit in log(N) steps.

### 10. Vector Search & RAG - Deeper Follow-ups

#### HNSW Deep Dive
**Q: How does HNSW (Hierarchical Navigable Small World) actually work?**
**A:** It creates a multi-layered graph. The top layer has very few nodes (long jumps). The bottom layer has all the vectors densely connected. A search starts at the top layer, greedily finding the closest node to the query, then drops down a layer and repeats. This allows incredibly fast traversal toward the true nearest neighbors, avoiding the need to compute distances to all points.

#### Chunking and Embeddings Deep Dive
**Q: In RAG, how does chunk size affect retrieval?**
**A:** If chunks are too small, they lose semantic context. The LLM might get a chunk like "It failed", but not know what "It" refers to. If chunks are too large, the embedding vector gets diluted (trying to represent too many ideas at once), leading to poor similarity matching, and it consumes too much context window.

#### LLM Evaluation Deep Dive
**Q: How did you evaluate the LLM outputs in your project?**
**A:** > **Needs candidate-specific confirmation** I used a framework to evaluate based on Faithfulness (is the answer derived only from the retrieved context?) and Answer Relevance (does it actually address the user's query?). We log these metrics over time to catch regressions when tweaking prompts or models.

### 11. System Design - Deep Dive

#### Load Balancing
**Q: How does a load balancer distribute traffic?**
**A:** Common algorithms include Round Robin (sequential distribution), Least Connections (routing to the server with the fewest active requests), and IP Hash (ensuring a specific user always hits the same server for sticky sessions).

#### Database Sharding
**Q: When scaling Postgres, how do you approach sharding?**
**A:** Sharding means splitting data across multiple independent databases based on a Shard Key (e.g., `user_id`). Queries that include the shard key route directly to one DB. Queries that don't must scatter-gather across all shards, which is incredibly slow. Designing a good shard key to avoid hot spots (one DB getting all the traffic) is the hardest part.

### 12. Security Basics
**Q: How do you prevent SQL Injection?**
**A:** Never use string formatting for SQL queries. Always use parameterized queries or an ORM like SQLAlchemy, which automatically sanitizes inputs by sending the query and parameters separately to the database engine.

**Q: How do you store passwords?**
**A:** Never in plain text. Always hash them using a strong, salted, computationally expensive algorithm like bcrypt or Argon2. This prevents attackers from reversing the hashes even if the database is compromised.
