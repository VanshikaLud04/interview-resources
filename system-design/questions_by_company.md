# System Design Questions — By Company

*Table of Contents*
- [Amazon](#amazon)
- [Anthropic](#anthropic)
- [Apple](#apple)
- [Brex](#brex)
- [ByteDance](#bytedance)
- [Chicago Trading Company](#chicago-trading-company)
- [Coinbase](#coinbase)
- [Databricks](#databricks)
- [Decagon](#decagon)
- [DoorDash](#doordash)
- [Figma](#figma)
- [Goldman Sachs](#goldman-sachs)
- [Google](#google)
- [Gusto](#gusto)
- [Harvey](#harvey)
- [Headway](#headway)
- [LinkedIn](#linkedin)
- [Lyft](#lyft)
- [Mercor](#mercor)
- [Meta](#meta)
- [Netflix](#netflix)
- [OpenAI](#openai)
- [Optiver](#optiver)
- [Pinterest](#pinterest)
- [Plaid](#plaid)
- [Ramp](#ramp)
- [Rippling](#rippling)
- [Salesforce](#salesforce)
- [Scale AI](#scale-ai)
- [Snowflake](#snowflake)
- [Stripe](#stripe)
- [Uber](#uber)
- [Walmart](#walmart)
- [Waymo](#waymo)
- [Zoox](#zoox)

---

### Amazon
**Design a Netflix-Style Streaming Platform**
- *Topics*: #video-streaming #cdn #high-availability #content-delivery
- *Difficulty*: 🔴 Very Hard
- *Hint*: Focus on the split between the control plane (user profiles, search, metadata) and data plane (video ingestion, encoding, delivery via CDN). You will need to discuss adaptive bitrate streaming, DRM, and global replication for low latency. Ensure you touch on the massive egress bandwidth requirements and caching strategies.

**Design an ML Training and Deployment Platform**
- *Topics*: #ml-infrastructure #orchestration #batch-processing #serving
- *Difficulty*: 🔴 Very Hard
- *Hint*: Break this down into data ingestion, model training pipelines (using DAG schedulers), model registry, and serving infrastructure. Discuss how to handle large datasets efficiently, checkpointing during training, and blue-green deployments for model updates.

**Design a Distributed Recurring Workflow Scheduler**
- *Topics*: #distributed-systems #task-scheduling #reliability
- *Difficulty*: 🟡 Hard
- *Hint*: Use a distributed database to store workflow metadata and a message queue or distributed timer system (like Time-Wheel) to trigger tasks. Ensure the scheduler is highly available and can handle node failures without missing or duplicating executions.

**Design a Financial Portfolio Dashboard**
- *Topics*: #real-time #caching #data-aggregation
- *Difficulty*: 🟢 Medium
- *Hint*: Leverage a real-time data streaming pipeline (e.g., Kafka) to ingest market updates. Use an in-memory cache (like Redis) for fast reads and a reliable database for historical data and user portfolios.

**Design Offline Reading-Progress Synchronization**
- *Topics*: #sync #offline-first #conflict-resolution
- *Difficulty*: 🟢 Medium
- *Hint*: Implement a local database on the client to store progress offline. Use conflict-free replicated data types (CRDTs) or logical timestamps (like Vector Clocks) to merge updates when the device comes back online.

**Design a Ticket Booking System**
- *Topics*: #concurrency #transactions #locking
- *Difficulty*: 🟡 Hard
- *Hint*: Discuss how to handle concurrent booking requests using optimistic or pessimistic locking in a relational database. Ensure transactional integrity when deducting inventory and processing payments.

**Global Music Streaming Platform**
- *Topics*: #audio-streaming #cdn #search
- *Difficulty*: 🟡 Hard
- *Hint*: Similar to video streaming, rely heavily on CDNs for audio delivery. Implement an efficient metadata search system using a search engine like Elasticsearch, and focus on the recommendation engine integration.

**Social News Feed With Live Engagement**
- *Topics*: #news-feed #pub-sub #real-time
- *Difficulty*: 🟡 Hard
- *Hint*: Use a fan-out architecture for distributing posts to followers' feeds. For live engagement (likes, comments), employ WebSockets and a pub/sub system to push updates to active clients instantly.

**Design an Online Ordering Platform**
- *Topics*: #e-commerce #inventory #microservices
- *Difficulty*: 🟢 Medium
- *Hint*: Design independent microservices for catalog, cart, checkout, and order management. Focus on idempotency for payments and reliable message delivery between services.

---

### Anthropic
**Design Peer-to-Peer Model Distribution**
- *Topics*: #p2p #large-files #ml-models
- *Difficulty*: 🔴 Very Hard
- *Hint*: Use a BitTorrent-like protocol to distribute massive model weights across a cluster of inference nodes. Discuss chunking, hashing for integrity, and the tracker architecture for peer discovery.

**Design a Model-Weight Deployment Platform**
- *Topics*: #mlops #storage #deployment
- *Difficulty*: 🟡 Hard
- *Hint*: Focus on an object storage backend optimized for large read throughput. Implement a versioning system and a staged rollout mechanism to safely deploy new model weights to production serving nodes.

**Design Opaque-GPU Inference Capacity**
- *Topics*: #resource-management #scheduling #gpu
- *Difficulty*: 🔴 Very Hard
- *Hint*: Abstract the underlying GPU hardware by providing a unified inference API. Design a multi-tenant scheduler that optimizes GPU utilization (e.g., batching, multiplexing) while ensuring isolation and QoS guarantees.

**Design Production ML Serving Observability**
- *Topics*: #observability #metrics #ml-monitoring
- *Difficulty*: 🟡 Hard
- *Hint*: Collect detailed metrics on request latency, token generation speed, and model errors. Use a time-series database (like Prometheus) and implement distributed tracing to track requests across the serving pipeline.

**Design a Prompt Playground**
- *Topics*: #web-app #llm-api #streaming
- *Difficulty*: 🟢 Medium
- *Hint*: Provide a responsive UI that connects to backend LLM endpoints. Focus on supporting Server-Sent Events (SSE) or WebSockets for streaming token responses and handling rate limiting gracefully.

---

### Apple
**Design a Mobile Application Event Collection API**
- *Topics*: #data-ingestion #high-throughput #batching
- *Difficulty*: 🟢 Medium
- *Hint*: Design an API gateway that accepts batches of events from mobile clients. Use a high-throughput message queue (e.g., Kafka) to buffer events before processing them asynchronously.

**Design an Analytical Data Dashboard**
- *Topics*: #olap #data-visualization #caching
- *Difficulty*: 🟡 Hard
- *Hint*: Use an OLAP database (like ClickHouse or Snowflake) for complex analytical queries. Implement materialized views or pre-aggregation pipelines for frequently accessed dashboards to ensure low-latency loads.

**Design a Multi-Device Photo Storage System**
- *Topics*: #storage #sync #media
- *Difficulty*: 🔴 Very Hard
- *Hint*: Separate the storage of metadata (in a scalable database) and media files (in object storage). Implement an efficient delta-sync mechanism and robust conflict resolution for multi-device updates.

**Design an Online Voting Service**
- *Topics*: #security #consistency #scalability
- *Difficulty*: 🟡 Hard
- *Hint*: Prioritize strong consistency and auditability, likely using a relational database with strict ACID properties or an immutable ledger. Discuss rate limiting and fraud detection to prevent abuse.

**Design an On-Prem Media Transfer for Cloud GPU Training**
- *Topics*: #hybrid-cloud #data-transfer #bandwidth-optimization
- *Difficulty*: 🟡 Hard
- *Hint*: Utilize an edge device or gateway to compress and encrypt data locally before transfer. Implement a resumable, parallelized upload client to maximize throughput over potentially unreliable WAN links.

**Design a Large JSON Transformation Service**
- *Topics*: #data-processing #streaming #scalability
- *Difficulty*: 🟢 Medium
- *Hint*: Process large JSON payloads using stream-based parsers (like SAX for JSON) rather than loading the entire object into memory. Design a scalable fleet of stateless workers to handle transformations concurrently.

---

### Brex
**Design a P2P Money Transfer System**
- *Topics*: #payments #transactions #idempotency
- *Difficulty*: 🟡 Hard
- *Hint*: Focus on ensuring transactional integrity and idempotency. Use a relational database or a dedicated ledger database to record debits and credits atomically.

---

### ByteDance
**Design a Multimedia Content Moderation Platform**
- *Topics*: #ml-inference #asynchronous-processing #content-delivery
- *Difficulty*: 🟡 Hard
- *Hint*: Build an event-driven pipeline where uploaded content triggers automated ML checks (text, image, video). Design a fallback mechanism for human review and ensure rapid processing to not delay content publication.

---

### Chicago Trading Company
**Design a Pre-Trade Risk Checking System**
- *Topics*: #low-latency #trading #in-memory-compute
- *Difficulty*: 🔴 Very Hard
- *Hint*: Ultra-low latency is critical. Keep risk parameters and current positions in-memory for microsecond-level checks. Discuss how to update these caches asynchronously without blocking trade execution.

---

### Coinbase
**Design a Brokerage Order Processing System**
- *Topics*: #financial-systems #matching-engine #event-sourcing
- *Difficulty*: 🔴 Very Hard
- *Hint*: Implement an event-sourced architecture to maintain an immutable log of all order state changes. Design a high-performance matching engine and discuss how to handle peak trading volumes reliably.

---

### Databricks
**Design an LLM Content Safety Service**
- *Topics*: #llm-guardrails #low-latency #text-processing
- *Difficulty*: 🟡 Hard
- *Hint*: This service sits in the critical path of LLM generation. Optimize for low latency by using fast classifiers or localized smaller models before invoking heavier safety checks.

**Online Bookseller Platform**
- *Topics*: #e-commerce #search #inventory
- *Difficulty*: 🟢 Medium
- *Hint*: Standard e-commerce architecture. Separate concerns into catalog, search (using Elasticsearch), cart, and order processing. Ensure strong consistency for inventory management.

**Chat API and Database System**
- *Topics*: #chat #nosql #real-time
- *Difficulty*: 🟡 Hard
- *Hint*: Use a NoSQL database (like Cassandra or DynamoDB) optimized for write-heavy workloads and time-series querying. Implement WebSockets for real-time message delivery and push notifications.

**Music Playlist System**
- *Topics*: #data-modeling #caching #read-heavy
- *Difficulty*: 🟢 Medium
- *Hint*: Model playlists effectively in a database to handle frequent reads and occasional updates. Use a caching layer to store popular playlists and reduce database load.

**Bookstore Batch Book-Price API**
- *Topics*: #api-design #batch-processing #caching
- *Difficulty*: 🟢 Medium
- *Hint*: Design an API that accepts bulk price update requests. Process these asynchronously using a message queue and background workers to update the database and invalidate relevant caches.

**WAL-Backed Batched Log-Writing System**
- *Topics*: #storage-engine #durability #performance
- *Difficulty*: 🔴 Very Hard
- *Hint*: Discuss the mechanics of a Write-Ahead Log (WAL). Explain how to batch writes for throughput while ensuring fsync durability to prevent data loss on crash.

**Design a Durable Local Event Writer**
- *Topics*: #event-logging #reliability #local-storage
- *Difficulty*: 🟡 Hard
- *Hint*: Write events locally to a durable file or lightweight database (like SQLite) first. Implement a background process to reliably forward these events to a central system with retries.

**Design a Chat Application Cache Architecture**
- *Topics*: #caching #invalidation #chat
- *Difficulty*: 🟡 Hard
- *Hint*: Cache active conversations and recent messages. Design an invalidation strategy that accurately reflects new messages, edits, or deletions in near real-time.

**Design a Chat Application with Message and Thread Deletion**
- *Topics*: #data-modeling #soft-delete #eventual-consistency
- *Difficulty*: 🟡 Hard
- *Hint*: Implement soft deletion using status flags in the database. Discuss how to propagate delete events to clients and eventually clean up the data asynchronously (tombstoning).

---

### Decagon
**Design a Multi-Tenant AI Gateway**
- *Topics*: #api-gateway #multi-tenancy #rate-limiting #routing
- *Difficulty*: 🟡 Hard
- *Hint*: Build a gateway that routes requests to different LLM providers based on tenant configuration. Focus on robust tenant isolation, token-aware rate limiting, and unified billing metrics.

---

### DoorDash
**Design a Scheduled Job Execution System**
- *Topics*: #scheduling #distributed-systems #cron
- *Difficulty*: 🟡 Hard
- *Hint*: Store job definitions and schedules in a reliable database. Use a distributed coordinator (like ZooKeeper or a leader-election mechanism) to assign jobs to workers at the specified times.

**Design a Downstream-Service Alert Notification System**
- *Topics*: #monitoring #alerting #pub-sub
- *Difficulty*: 🟢 Medium
- *Hint*: Ingest metrics and logs, evaluate alerting rules, and dispatch notifications. Use deduplication and grouping to prevent alert fatigue.

**Design a Three-Day Charity Event System**
- *Topics*: #spiky-traffic #scalability #donations
- *Difficulty*: 🟡 Hard
- *Hint*: Anticipate massive traffic spikes during the short event window. Pre-scale infrastructure and heavily cache read-only data. Ensure the donation processing path is highly resilient and decoupled from the main UI.

---

### Figma
**Design Comments for a Collaborative Canvas**
- *Topics*: #real-time #spatial-data #collaboration
- *Difficulty*: 🟡 Hard
- *Hint*: Use spatial indexing to efficiently query comments based on viewport coordinates. Implement real-time updates using WebSockets so all collaborators see new comments instantly.

**Design Trending Design Files**
- *Topics*: #trending #analytics #stream-processing
- *Difficulty*: 🟢 Medium
- *Hint*: Process view and interaction events in a stream-processing framework (like Flink or Spark Streaming) to calculate trending scores over sliding time windows.

**Design Access-Aware Design-File Retrieval and Ranking**
- *Topics*: #search #authorization #ranking
- *Difficulty*: 🟡 Hard
- *Hint*: Integrate Access Control Lists (ACLs) directly into the search index (e.g., Elasticsearch) to ensure users only see files they have permission to access, while maintaining fast query and ranking performance.

**Design an Agent Evaluation Platform**
- *Topics*: #ai-agents #evaluation #batch-processing
- *Difficulty*: 🟡 Hard
- *Hint*: Build a system to run multiple versions of AI agents against a comprehensive test suite of tasks. Focus on managing asynchronous execution, capturing detailed traces, and providing comparative analytics.

---

### Goldman Sachs
**Design a Live-Stream Chat System**
- *Topics*: #chat #high-throughput #broadcasting
- *Difficulty*: 🟡 Hard
- *Hint*: Handle massive fan-out for popular streams. Batch chat messages on the server side and send them to clients at a controlled rate to prevent overwhelming the UI.

---

### Google
**Design a Real-Time Anomaly Detection Service**
- *Topics*: #stream-processing #ml-inference #monitoring
- *Difficulty*: 🔴 Very Hard
- *Hint*: Ingest high-volume telemetry data via Kafka. Use stream processing engines to calculate moving averages or run lightweight ML models in real-time to flag anomalies.

**Distributed Quoted-Log Tokenization**
- *Topics*: #data-processing #distributed-algorithms #parsing
- *Difficulty*: 🔴 Very Hard
- *Hint*: Design an algorithm to parallelize parsing of massive log files, correctly handling quoted strings that might span chunk boundaries. Requires deep understanding of MapReduce-style processing.

**Ranked Deals Discovery and Claim Service**
- *Topics*: #search #ranking #concurrency
- *Difficulty*: 🟡 Hard
- *Hint*: Combine a search and ranking engine for discovery with a highly concurrent, transactional system for claiming limited-quantity deals to prevent overselling.

---

### Gusto
**Design a Restaurant Staff Scheduling System**
- *Topics*: #scheduling #calendar #web-app
- *Difficulty*: 🟢 Medium
- *Hint*: Focus on the data model for shifts, employees, and availability. Handle constraints and overlaps, and design an efficient API for generating and publishing schedules.

---

### Harvey
**Design a Grounded Document-Vault QA**
- *Topics*: #rag #vector-db #security
- *Difficulty*: 🟡 Hard
- *Hint*: Design a Retrieval-Augmented Generation (RAG) architecture. Emphasize strict document-level access control during the vector retrieval phase to ensure users only query against authorized documents.

---

### Headway
**Design Vacation Rental Listing Search**
- *Topics*: #search #geospatial #filtering
- *Difficulty*: 🟢 Medium
- *Hint*: Use a search engine with geospatial capabilities (like Elasticsearch) to index properties. Optimize for complex, faceted queries combining location, dates, amenities, and price.

---

### LinkedIn
**CI/CD Pipeline Orchestration System**
- *Topics*: #orchestration #dag #event-driven
- *Difficulty*: 🟡 Hard
- *Hint*: Model pipelines as Directed Acyclic Graphs (DAGs). Build a robust worker pool to execute build/test jobs, manage dependencies between stages, and handle job failures or retries gracefully.

---

### Lyft
**Donation Platform**
- *Topics*: #payments #microservices #event-driven
- *Difficulty*: 🟢 Medium
- *Hint*: Similar to general payment processing, focus on idempotency, reliable integration with payment gateways (like Stripe), and generating audit trails for tax purposes.

---

### Mercor
**Design a Unique Username Registration System**
- *Topics*: #concurrency #database #caching
- *Difficulty*: 🟢 Medium
- *Hint*: Ensure strict uniqueness under high concurrency. Use unique constraints in a relational database and consider Bloom filters or caching layers to quickly reject unavailable usernames.

**Design a Durable Work Orchestration Platform**
- *Topics*: #workflow #state-machine #reliability
- *Difficulty*: 🟡 Hard
- *Hint*: Design a distributed state machine (similar to AWS Step Functions). Ensure that the state of each workflow step is durably persisted so execution can resume from failures.

---

### Meta
**Design a Multimodal Public-Content Safety Platform**
- *Topics*: #content-moderation #ml-inference #high-throughput
- *Difficulty*: 🔴 Very Hard
- *Hint*: Scale to handle billions of multimodal posts. Route text, images, and videos to specialized ML models. Design a tiered approach where high-confidence flags are auto-moderated and edge cases go to human review.

---

### Netflix
**Design a Grounded Recommendation Chatbot**
- *Topics*: #rag #recommendation-systems #conversational-ai
- *Difficulty*: 🟡 Hard
- *Hint*: Combine user viewing history, catalog metadata, and conversational context into a prompt. Use RAG techniques to ensure the chatbot recommends actual, available titles.

**Design Advertising Order Lifecycle Tracking**
- *Topics*: #ad-tech #workflow #state-tracking
- *Difficulty*: 🟡 Hard
- *Hint*: Track complex ad campaigns from creation to delivery. Use an event-sourced system to maintain a complete history of campaign state changes for billing and auditing.

**Design an Advertising Frequency-Capping Service**
- *Topics*: #ad-tech #distributed-counters #low-latency
- *Difficulty*: 🔴 Very Hard
- *Hint*: Ensure users don't see the same ad too many times. Implement high-throughput, low-latency distributed counters (e.g., using Redis) that can handle millions of read/write operations per second.

**Design a Machine-Learning Job Scheduler**
- *Topics*: #mlops #resource-allocation #scheduling
- *Difficulty*: 🟡 Hard
- *Hint*: Schedule complex ML jobs across a cluster of compute nodes. Manage dependencies, handle heterogeneous hardware (CPU/GPU), and implement fair-share or priority-based scheduling algorithms.

**Design a Personalized Recommendation System**
- *Topics*: #recommendation #machine-learning #batch-and-real-time
- *Difficulty*: 🔴 Very Hard
- *Hint*: Discuss a two-stage approach: candidate generation (batch processing) and ranking (real-time). Explain how to manage the vast feature stores needed for personalization.

---

### OpenAI
**Design a Distributed Crossword Solver**
- *Topics*: #distributed-algorithms #search-space #parallel-processing
- *Difficulty*: 🔴 Very Hard
- *Hint*: Divide the search space of the crossword puzzle. Use a master-worker architecture where workers solve sub-grids and communicate constraints back to the master to coordinate the global solution.

**Design an Online Chess Platform**
- *Topics*: #gaming #real-time #state-management
- *Difficulty*: 🟡 Hard
- *Hint*: Use WebSockets for real-time move transmission. Maintain authoritative game state on the server to prevent cheating. Design an efficient matchmaking system based on player ratings.

**Design a Distributed Video Generation Platform**
- *Topics*: #gen-ai #video-processing #heavy-compute
- *Difficulty*: 🔴 Very Hard
- *Hint*: Video generation is extremely compute-intensive. Distribute frame generation tasks across a massive GPU cluster, manage intermediate storage, and design a robust final assembly pipeline.

**Payment Hold Charge and Settlement**
- *Topics*: #payments #state-machine #financial-consistency
- *Difficulty*: 🟡 Hard
- *Hint*: Model the lifecycle of a payment authorization, hold, capture, and settlement. Ensure reliable communication with banking APIs and handle edge cases like hold expirations.

**Design a Stateless Generative AI Chat Service**
- *Topics*: #llm-serving #stateless #context-management
- *Difficulty*: 🟢 Medium
- *Hint*: Since the service is stateless, the client must send the entire conversation history with each request. Focus on optimizing API endpoints and effectively managing token limits.

**Design a Declarative Infrastructure Orchestrator**
- *Topics*: #infrastructure #state-reconciliation #control-plane
- *Difficulty*: 🔴 Very Hard
- *Hint*: Similar to Kubernetes. Accept declarative intent (target state) and build a controller loop that continuously compares actual state with target state, issuing commands to reconcile them.

**Design an Agent Evaluation Platform**
- *Topics*: #evaluation #testing #ai-agents
- *Difficulty*: 🟡 Hard
- *Hint*: Create isolated environments (sandboxes) for agents to execute tasks. Build a framework to collect metrics, logs, and compare performance against benchmark datasets.

**Design an Online Payment Processing System**
- *Topics*: #payments #security #high-availability
- *Difficulty*: 🟡 Hard
- *Hint*: Focus on PCI compliance, secure tokenization of credit card data, and a highly available architecture that can withstand regional failures without dropping transactions.

---

### Optiver
**Design a News Subscription Processing System**
- *Topics*: #trading #news-feed #low-latency
- *Difficulty*: 🟡 Hard
- *Hint*: Ingest raw news feeds, parse and normalize them at high speed, and distribute actionable signals to trading algorithms with minimal latency.

---

### Pinterest
**Design an Ad Event Measurement Platform**
- *Topics*: #ad-tech #analytics #data-pipeline
- *Difficulty*: 🟡 Hard
- *Hint*: Ingest massive volumes of clicks, views, and conversions. Build a robust ETL pipeline to aggregate these events and attribute them to specific ad campaigns for reporting.

**Design Viral Comment Notification Aggregation**
- *Topics*: #notifications #aggregation #batching
- *Difficulty*: 🟡 Hard
- *Hint*: Avoid spamming users with notifications when a post goes viral. Implement a buffering and aggregation system that groups notifications ("User A, B, and 100 others commented") before dispatching them.

**Design an Inventory Management Service**
- *Topics*: #e-commerce #consistency #transactions
- *Difficulty*: 🟡 Hard
- *Hint*: Handle high-concurrency read/write operations for product inventory. Use techniques like inventory reservation and sagas or two-phase commit across microservices.

**Design a Real-Time Category Leaderboard**
- *Topics*: #leaderboard #real-time #redis
- *Difficulty*: 🟢 Medium
- *Hint*: Utilize Redis Sorted Sets to maintain leaderboards efficiently. Discuss how to partition the data if the leaderboard grows too large for a single Redis instance.

---

### Plaid
**Design an Embedded Pay-by-Bank Checkout**
- *Topics*: #fintech #api-integration #security
- *Difficulty*: 🟡 Hard
- *Hint*: Design an embeddable widget that securely authenticates users with their bank. Manage the flow of OAuth tokens, handle webhook callbacks from banking partners, and ensure transactional security.

---

### Ramp
**Design a Real-Time Payment Count Dashboard**
- *Topics*: #analytics #stream-processing #dashboard
- *Difficulty*: 🟢 Medium
- *Hint*: Use stream processing (e.g., Kafka Streams) to aggregate payment counts in real-time. Push updates to a live dashboard using WebSockets or Server-Sent Events.

---

### Rippling
**Design an Event Ingestion Platform**
- *Topics*: #data-ingestion #kafka #scalability
- *Difficulty*: 🟡 Hard
- *Hint*: Build a highly scalable API to receive events from various sources. Buffer them in Kafka, and design flexible consumer groups to route data to different downstream sinks (data lake, real-time analytics).

**Design a Frontend News Feed with Offline Virtualization**
- *Topics*: #frontend #offline-first #caching
- *Difficulty*: 🟡 Hard
- *Hint*: Focus on the client-side architecture. Use IndexedDB for local storage, implement aggressive caching, and build a service worker to intercept requests and serve content offline seamlessly.

**Design a Personalized News Aggregation Platform**
- *Topics*: #news #recommendation #content-delivery
- *Difficulty*: 🟡 Hard
- *Hint*: Ingest news from various RSS feeds or APIs. Extract entities and categorize articles. Build a recommendation engine that matches user profiles with categorized news content.

**Design a Publisher News Feed**
- *Topics*: #news-feed #fan-out #caching
- *Difficulty*: 🟡 Hard
- *Hint*: Similar to social feeds. Use a hybrid fan-out approach: push model for active users, pull model for inactive users. Heavily cache the top stories.

**Design an Exact-Room Hotel Reservation System**
- *Topics*: #booking #inventory #locking
- *Difficulty*: 🟡 Hard
- *Hint*: Managing exact rooms (e.g., Room 101) requires precise inventory tracking and locking mechanisms to prevent double-booking specific physical assets, unlike general category booking.

**Design a Room-Type Hotel Booking System**
- *Topics*: #booking #inventory #scalability
- *Difficulty*: 🟢 Medium
- *Hint*: Easier than exact-room booking. Track total counts of available room types per date. Use optimistic concurrency control when decrementing available inventory.

**Design a Centralized Log Ingestion and Search Platform**
- *Topics*: #logging #search #elk-stack
- *Difficulty*: 🟡 Hard
- *Hint*: Design a system similar to Splunk or ELK. Use a message queue to buffer logs, logstash-like workers for parsing/normalization, and a search engine (Elasticsearch) for indexing and querying.

---

### Salesforce
**Design an Event-Driven Notification System**
- *Topics*: #notifications #event-driven #routing
- *Difficulty*: 🟢 Medium
- *Hint*: Build a central notification bus. Services publish generic events, and a rules engine determines which users should be notified and via which channels (email, SMS, push).

**Design a Rental Car Reservation System**
- *Topics*: #booking #inventory #state-machine
- *Difficulty*: 🟡 Hard
- *Hint*: Track the complex lifecycle of a rental car (available, reserved, rented, maintenance). Design an efficient search system for available cars by location and date.

**Design a Third-Party SaaS Integration Platform**
- *Topics*: #integration #api-management #webhooks
- *Difficulty*: 🟡 Hard
- *Hint*: Manage authentication (OAuth), rate limits, and data mapping for external APIs. Implement robust webhook handling and background job queues for data syncing.

---

### Scale AI
**Design an Embedding and Classification API**
- *Topics*: #ml-serving #embeddings #vector-db
- *Difficulty*: 🟡 Hard
- *Hint*: Expose endpoints for generating vector embeddings from text/images. Integrate a Vector Database (like Pinecone or Milvus) to allow fast similarity search and classification based on those embeddings.

---

### Snowflake
**Design a Fault-Tolerant Cloud Queue Service**
- *Topics*: #message-queue #distributed-systems #durability
- *Difficulty*: 🔴 Very Hard
- *Hint*: Design an SQS clone. Focus on achieving high availability and durability. Discuss partitioning, replication strategies, and how to handle visibility timeouts and dead-letter queues.

**Design a Unified SaaS Analytics Platform**
- *Topics*: #analytics #olap #multi-tenancy
- *Difficulty*: 🔴 Very Hard
- *Hint*: Build a multi-tenant data warehouse architecture. Discuss data separation strategies, efficient querying over massive partitioned datasets, and resource isolation between tenants.

---

### Stripe
**Design an Idempotent Ledger Service**
- *Topics*: #fintech #ledger #idempotency #acid
- *Difficulty*: 🔴 Very Hard
- *Hint*: The core of any payment system. Design a highly consistent double-entry ledger. Emphasize strict idempotency handling using unique request keys and robust database transactions.

---

### Uber
**Design a Driver Review Leaderboard**
- *Topics*: #leaderboard #analytics #caching
- *Difficulty*: 🟢 Medium
- *Hint*: Aggregate review scores periodically or via stream processing. Use a caching layer to serve leaderboard views rapidly, segmenting by region or vehicle type.

**Design a Driver Payout System**
- *Topics*: #payments #batch-processing #reliability
- *Difficulty*: 🟡 Hard
- *Hint*: Build a reliable batch processing pipeline that calculates earnings based on rides, promotions, and fees, and initiates secure transfers to driver bank accounts on a set schedule.

**Design a Price Movement Alert Platform**
- *Topics*: #alerting #stream-processing #high-throughput
- *Difficulty*: 🟡 Hard
- *Hint*: Ingest real-time price streams. Evaluate millions of user-defined alert conditions continuously. Optimize the evaluation engine to prevent bottlenecks.

**Rolling Stock Price Alert System**
- *Topics*: #finance #stream-processing #complex-event-processing
- *Difficulty*: 🟡 Hard
- *Hint*: Similar to price movement alerts but requires evaluating complex conditions over time windows (e.g., "moving average drops by 5%"). Utilize Complex Event Processing (CEP) frameworks.

---

### Walmart
**Design a Marketplace Product Catalog Platform**
- *Topics*: #e-commerce #catalog #search
- *Difficulty*: 🟡 Hard
- *Hint*: Handle product data from multiple third-party sellers. Design a system to normalize attributes, manage variations (size, color), and power a highly responsive product search and discovery experience.

---

### Waymo
**Design a Global Small-Image Cache**
- *Topics*: #caching #cdn #image-delivery
- *Difficulty*: 🟢 Medium
- *Hint*: Design a distributed cache hierarchy (edge caches, regional caches). Optimize for high read throughput and discuss cache eviction policies (like LRU) suitable for small assets.

**Design a Matchmaking Service**
- *Topics*: #dispatch #geospatial #algorithms
- *Difficulty*: 🟡 Hard
- *Hint*: Match riders with autonomous vehicles efficiently. Use geospatial indexing to find nearby available vehicles and implement assignment algorithms that optimize for overall system efficiency (e.g., minimizing wait times).

---

### Zoox
**Design a Real-Time Fleet Location Display**
- *Topics*: #geospatial #real-time #websockets
- *Difficulty*: 🟡 Hard
- *Hint*: Ingest high-frequency location pings from vehicles. Use spatial indexing and a publish-subscribe model to push relevant updates only to users viewing a specific map area.

**Design a Nearby Autonomous Vehicle Search Service**
- *Topics*: #search #geospatial #indexing
- *Difficulty*: 🟢 Medium
- *Hint*: Implement a fast geospatial query system (using Geohashes, Quadtrees, or H3 indexes). Ensure the index can handle high update rates as vehicle positions change constantly.
