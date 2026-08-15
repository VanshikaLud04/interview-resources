# Resume Claim Map — Vanshika Ludhani

Every technical claim on the resume, decomposed into what an interviewer can drill into.

---

## Project 1: RAGOS

### Bullet 1: "Built a framework-agnostic experimentation and optimization platform for RAG, enabling automated evaluation and optimization of retrieval pipelines under production constraints."

```
Claim: "framework-agnostic"
 ├── What does framework-agnostic mean here?
 ├── How is it framework-agnostic? (interfaces.py → abstract base classes)
 ├── What frameworks could plug in?
 ├── What design pattern enables this? (Strategy/Plugin)
 ├── Why is framework-agnosticism important?
 └── What would break if it weren't framework-agnostic?

Claim: "experimentation and optimization platform"
 ├── What is being experimented on? (RAG pipeline configurations)
 ├── What is being optimized? (quality, cost, latency)
 ├── How does experimentation work? (submit config → run pipeline → evaluate → compare)
 ├── How does optimization work? (constraint-aware random search)
 ├── What makes this a "platform" vs a "script"? (API, persistence, plugin system)
 └── How do experiments persist and compare? (PostgreSQL)

Claim: "RAG"
 ├── What is RAG?
 ├── Why RAG vs fine-tuning?
 ├── RAG pipeline components (chunking → embedding → retrieval → generation)
 ├── What are the tunable parameters in a RAG pipeline?
 ├── What makes RAG hard to optimize?
 ├── How do you evaluate RAG quality?
 └── What are common RAG failure modes? (irrelevant retrieval, hallucination, context window overflow)

Claim: "automated evaluation"
 ├── What metrics are computed? (faithfulness, relevance, answer correctness)
 ├── How is evaluation automated? (LLM-as-judge)
 ├── What is the evaluation pipeline?
 ├── How reliable is automated evaluation?
 ├── What are the alternatives? (human eval, BLEU/ROUGE)
 └── How do you handle non-determinism in LLM evaluation?

Claim: "production constraints"
 ├── What constraints? (latency, cost, local-only hardware)
 ├── How are constraints expressed? (declarative: latency <= 500ms)
 ├── How are constraints enforced during optimization?
 ├── What happens if no configuration satisfies all constraints?
 └── How do constraints interact with quality objectives?
```

### Bullet 2: "Implemented constraint-aware Random Search to optimize RAG pipelines (chunking, retrieval, generation) under latency and cost constraints, evaluating 100+ configurations."

```
Claim: "constraint-aware Random Search"
 ├── What is Random Search?
 ├── Why Random Search over Grid Search?
 ├── Why Random Search over Bayesian Optimization?
 ├── What makes it "constraint-aware"? (repair/filter configurations that violate constraints)
 ├── How does the search space look? (chunk_size × retrieval_strategy × generator_model × ...)
 ├── What is the dimensionality of the search space?
 ├── How are configurations sampled?
 ├── How are invalid configurations handled?
 └── What stopping criteria exist?

Claim: "chunking, retrieval, generation"
 ├── What chunking strategies exist? (fixed-size, recursive, semantic)
 ├── What retrieval strategies exist? (dense, sparse, hybrid)
 ├── What generation models are supported? (Gemini, Ollama/local)
 ├── How does changing chunk size affect quality?
 ├── How does changing retrieval strategy affect latency?
 └── What is the interaction between chunking and retrieval?

Claim: "latency and cost constraints"
 ├── How is latency measured? (end-to-end pipeline time)
 ├── How is cost measured? (LiteLLM token cost profiling)
 ├── What units? (ms, USD per query)
 ├── How are constraints checked before vs after trial execution?
 └── What is the cost of running 100+ evaluations?

Claim: "100+ configurations"
 ├── What counts as one "configuration"?
 ├── How long does one configuration take to evaluate?
 ├── Total experiment runtime?
 ├── How are results stored? (PostgreSQL)
 ├── How are results compared? (Pareto frontier)
 └── Is 100+ configurations a lot or typical for this search space?
```

### Bullet 3: "Implemented async experiment orchestration with automated evaluation, LiteLLM token cost profiling, and Pareto-frontier pipeline selection."

```
Claim: "async experiment orchestration"
 ├── What is async here? (Python asyncio)
 ├── Why async vs synchronous sequential execution?
 ├── How many experiments run concurrently?
 ├── What is the orchestration flow? (sample config → execute pipeline → evaluate → store result)
 ├── What manages the lifecycle of experiments?
 ├── How do you handle experiment failures?
 └── What happens if an experiment hangs?

Claim: "LiteLLM token cost profiling"
 ├── What is LiteLLM?
 ├── Why LiteLLM over direct API calls?
 ├── How does token cost profiling work?
 ├── What pricing data does LiteLLM use?
 ├── How accurate is the cost estimation?
 └── How does cost feed into the optimization objective?

Claim: "Pareto-frontier pipeline selection"
 ├── What is a Pareto frontier?
 ├── What dimensions? (quality, cost, latency)
 ├── How is Pareto dominance computed?
 ├── Why Pareto instead of a weighted score?
 ├── How does the user pick from the Pareto set?
 ├── What is non-dominated sorting?
 └── What happens when most solutions are Pareto-optimal (too many)?
```

### Bullet 4: "Designed a plugin-based architecture enabling new retrievers, embedders, generators, and evaluators without modifying the core optimization engine."

```
Claim: "plugin-based architecture"
 ├── What design pattern? (Strategy + Registry/Factory)
 ├── How are plugins registered? (plugins.py)
 ├── How are plugins discovered? (explicit registration or auto-discovery?)
 ├── What is the plugin interface? (interfaces.py → abstract base classes)
 ├── Can plugins be added at runtime?
 ├── How does the optimizer use plugins without knowing concrete types?
 └── What are the interface methods for each plugin type?

Claim: "without modifying the core optimization engine"
 ├── What is the Open/Closed Principle?
 ├── How does this architecture satisfy OCP?
 ├── What would need to change if a new embedding dimension were added?
 ├── What if a new evaluation metric is needed?
 └── Give an example of adding a new retriever (step by step).
```

### RAGOS Technology Map
```
RAGOS
 ├── Python (asyncio, type hints, abstract classes)
 ├── FastAPI (REST API, dependency injection, async endpoints)
 ├── PostgreSQL (experiment storage, metadata, results)
 ├── ChromaDB (vector storage, similarity search)
 ├── LiteLLM (multi-provider LLM abstraction, cost tracking)
 ├── Docker (docker-compose: app + postgres + chroma)
 ├── AsyncIO (experiment orchestration, non-blocking I/O)
 ├── GitHub Actions (CI pipeline)
 └── Concepts
      ├── RAG (retrieval-augmented generation)
      ├── Vector Search (cosine similarity, ANN)
      ├── Random Search (hyperparameter optimization)
      ├── Pareto Frontier (multi-objective optimization)
      ├── Plugin Architecture (Strategy pattern, OCP)
      ├── Information Retrieval (precision, recall, NDCG)
      └── LLM Evaluation (LLM-as-judge, faithfulness)
```

---

## Project 2: LLM Cost Guard

### Bullet 1: "Implemented atomic Redis token reservation to eliminate double charging under concurrent requests, validated with 7,100+ concurrent integration tests."

```
Claim: "atomic Redis token reservation"
 ├── What is atomicity in this context?
 ├── Why is atomicity needed? (concurrent requests)
 ├── What is the TOCTOU problem?
 ├── How does Redis achieve atomicity? (Lua scripts → single-threaded execution)
 ├── What does the Lua script do exactly? (check budget → reserve → return)
 ├── Why Lua script vs WATCH/MULTI/EXEC pipeline?
 ├── Why Lua script vs Redis transactions?
 ├── What Redis data structures are used? (hashes? sorted sets? strings?)
 ├── What is the key schema? (org:budget:minute, org:budget:hour, etc.)
 ├── What happens if Redis crashes mid-script?
 └── Is the Lua script idempotent?

Claim: "eliminate double charging"
 ├── What is double charging in this context?
 ├── How does double charging happen without atomicity?
 ├── Walk through a race condition scenario
 ├── How does reserve-and-reconcile prevent it?
 ├── What is "reservation" vs "actual usage"?
 ├── What happens after the LLM call completes? (reconciliation)
 ├── What if the LLM call fails after reservation? (release reservation)
 ├── What if the reconciliation fails? (reservation expires? leaked budget?)
 └── How do you handle reservation timeout/cleanup?

Claim: "7,100+ concurrent integration tests"
 ├── What tool was used? (Locust — locustfile.py exists in repo)
 ├── What was the test setup? (concurrent users, spawn rate, duration)
 ├── What was being validated? (no budget overruns, correct totals, no double-charge)
 ├── How do you verify correctness after 7,100 requests? (final budget = initial - actual sum)
 ├── What failure modes were discovered?
 ├── What is the p50/p95/p99 latency?
 ├── Were there any failures? (loadtest_results_failures.csv exists — empty = no failures)
 ├── Is "7,100+ concurrent integration tests" = 7,100 simultaneous requests or 7,100 total requests in load test?
 └── Could you reproduce this test? Walk me through running it.
```

### Bullet 2: "Built a production AI Gateway for centralized policy enforcement, intelligent provider routing, and token budget management across multiple LLM providers."

```
Claim: "production AI Gateway"
 ├── What is an AI Gateway?
 ├── How is this different from a simple proxy?
 ├── What does "production" mean here? (Has this been deployed in prod? → Needs candidate confirmation)
 ├── What is the request lifecycle through the gateway?
 └── Why centralize instead of per-service rate limiting?

Claim: "centralized policy enforcement"
 ├── What policies are enforced? (model allowlists, token limits, org restrictions)
 ├── Where are policies stored? (PostgreSQL)
 ├── How are policies evaluated? (policy.py)
 ├── What happens when a policy is violated? (403)
 ├── Can policies be updated at runtime?
 └── How do policies interact with multi-tenancy?

Claim: "intelligent provider routing"
 ├── What providers are supported? (OpenAI, Anthropic, Groq)
 ├── What makes routing "intelligent"? (cost, health, latency, capability filters)
 ├── What is the routing pipeline? (Policy → Capability → Health → Cost → Latency → Select)
 ├── How is provider health tracked? (circuit breaker)
 ├── What is the fallback if the preferred provider is down?
 ├── How is cost computed for routing decisions?
 └── How does routing handle model-specific capabilities? (not all providers support all models)

Claim: "token budget management"
 ├── What is a token budget? (max tokens per org per time window)
 ├── What time windows? (minute, hour, day)
 ├── How is the budget tracked? (Redis)
 ├── How is the budget enforced? (reserve before call, reconcile after)
 ├── What is reserve-and-reconcile vs sliding window?
 └── How accurate is token estimation before the call?

Claim: "multiple LLM providers"
 ├── How many providers? (3: OpenAI, Anthropic, Groq)
 ├── How is the provider abstraction implemented? (LLMProvider interface → providers.py)
 ├── What methods does the interface define?
 ├── How do you add a new provider?
 ├── How do you handle different token counting across providers?
 └── How do you handle different API formats?
```

### Bullet 3: "Scaled request handling via async task queues (RabbitMQ/Celery), isolating latency-sensitive API paths from persistence workloads with a common provider interface."

```
Claim: "async task queues (RabbitMQ/Celery)"
 ├── What is Celery? What is RabbitMQ?
 ├── Why a task queue here?
 ├── What tasks are async? (persistence, logging, reconciliation)
 ├── What tasks are synchronous? (auth, policy check, LLM call, response)
 ├── Why RabbitMQ as broker vs Redis as broker?
 ├── How are Celery workers configured?
 ├── What happens if a Celery worker crashes?
 ├── What is the retry policy?
 ├── What about dead letter queues?
 └── How do you monitor Celery workers?

Claim: "isolating latency-sensitive API paths from persistence workloads"
 ├── What is latency-sensitive? (the LLM call + response to client)
 ├── What is persistence workload? (writing usage logs, updating budgets in PostgreSQL)
 ├── Why isolate them? (don't make the user wait for DB writes)
 ├── How are they isolated? (Celery task fires after response, not before)
 ├── What if the persistence task fails? (data loss? retry?)
 └── Is there eventual consistency here?

Claim: "common provider interface"
 ├── What is the interface? (LLMProvider abstract class)
 ├── What methods? (complete, stream, health_check, estimate_tokens)
 ├── How is dependency injection done? (FastAPI Depends)
 ├── What pattern? (Strategy pattern + Dependency Injection)
 └── How does the router use the interface without knowing concrete types?
```

### LLM Cost Guard Technology Map
```
LLM Cost Guard
 ├── Python
 ├── FastAPI (gateway API, middleware, dependency injection)
 ├── PostgreSQL (policies, usage logs, auth, orgs)
 ├── Redis (token reservation, rate limiting, circuit breaker state, semantic cache)
 │    ├── Lua Scripts (atomic reservation)
 │    ├── Data Structures (hashes, strings, sorted sets)
 │    ├── RediSearch (semantic cache vector index)
 │    └── Pub/Sub or Streams (if used)
 ├── Celery (async task workers)
 ├── RabbitMQ (message broker for Celery)
 ├── Docker (docker-compose: app + redis + postgres + rabbitmq + celery worker)
 ├── Kubernetes (deployment — depth needs confirmation)
 ├── AWS (deployment — depth needs confirmation)
 ├── Locust (load testing — locustfile.py)
 ├── GitHub Actions (CI pipeline)
 └── Concepts
      ├── TOCTOU (time-of-check-time-of-use race condition)
      ├── Reserve-and-Reconcile (two-phase token accounting)
      ├── Circuit Breaker (closed → open → half-open)
      ├── Semantic Caching (vector similarity for cached responses)
      ├── Multi-tenancy (org isolation, hierarchical policies)
      ├── Provider Abstraction (Strategy pattern)
      ├── Rate Limiting (token-based, time-windowed)
      ├── Dependency Injection (FastAPI Depends)
      └── SSE/Streaming (non-blocking streaming telemetry)
```

---

## Project 3: Focus Lock

### Bullet 1: "Developed a privacy-first Edge AI analytics platform performing on-device distraction detection with 97.2% recall and <30 ms inference latency."

```
Claim: "privacy-first"
 ├── What makes it privacy-first? (all processing on-device, no cloud)
 ├── Is any data sent externally? (no)
 ├── Where is data stored? (SQLite, local disk)
 ├── Why is privacy important for this use case?
 └── What would change if you added cloud sync?

Claim: "Edge AI"
 ├── What is Edge AI?
 ├── Why edge vs cloud inference?
 ├── What are the constraints? (CPU, memory, battery, latency)
 ├── What model optimization techniques apply? (quantization, pruning)
 └── How does this differ from cloud ML?

Claim: "on-device distraction detection"
 ├── What distractions are detected? (phone, eating, drinking, talking)
 ├── What model? (YOLOv8)
 ├── What is YOLOv8? How does it work? (anchor-free, single-shot detection)
 ├── What input does the model take? (webcam frames)
 ├── What output? (bounding boxes + class + confidence)
 ├── How is gaze direction tracked? (MediaPipe head-pose estimation)
 ├── How are YOLO + MediaPipe signals combined?
 └── What preprocessing is done on frames?

Claim: "97.2% recall"
 ├── What is recall? (true positives / (true positives + false negatives))
 ├── Recall of WHAT? (distraction detection overall? phone specifically?)
 ├── What dataset was this measured on? (benchmarks/eval_metrics.py)
 ├── How many samples?
 ├── What is the precision? (unknown from resume — needs confirmation)
 ├── What is the F1 score?
 ├── Is this on the training set or a holdout set?
 ├── How was the evaluation conducted?
 └── What are the failure modes? (false negatives = missed distractions)

Claim: "<30 ms inference latency"
 ├── What hardware? (MacBook? What CPU/GPU?)
 ├── What model size? (YOLOv8n? YOLOv8s?)
 ├── Is this per-frame latency? (yes, inference only, not capture)
 ├── How was this measured? (benchmarks/latency_profile.py)
 ├── What is the end-to-end frame processing time? (capture + preprocess + inference + postprocess)
 ├── What would latency be on a Raspberry Pi?
 └── Does this include MediaPipe gaze estimation? Or only YOLO?
```

### Bullet 2: "Designed an event-driven producer-consumer pipeline separating capture, inference, telemetry, and analytics using lock-safe async queues and an in-memory EventBus."

```
Claim: "event-driven producer-consumer pipeline"
 ├── What is a producer-consumer pattern?
 ├── What are the producers? (camera capture, inference engine)
 ├── What are the consumers? (dashboard, SQLite writer, analytics engine)
 ├── Why producer-consumer vs direct function calls?
 ├── What queue implementation? (Python queue.Queue — thread-safe)
 ├── Why threads instead of asyncio? (OpenCV is blocking, CPU-bound inference)
 └── How many threads? What does each do?

Claim: "separating capture, inference, telemetry, and analytics"
 ├── Camera Thread → Inference Thread → Attention Engine → EventBus
 ├── Why separate capture from inference? (camera I/O-bound, inference CPU-bound)
 ├── What is telemetry here? (resource usage: CPU, RAM, FPS)
 ├── What is analytics here? (historical trends, heatmaps, timelines)
 ├── How does separation improve performance?
 └── What happens if the inference thread is slower than capture? (queue grows, frames dropped?)

Claim: "lock-safe async queues"
 ├── What makes them "lock-safe"? (Python queue.Queue is internally locked)
 ├── Why is thread safety needed?
 ├── What would happen without locks? (race conditions, corrupted data)
 ├── What lock mechanism? (mutex inside queue.Queue)
 └── Any deadlock risks?

Claim: "in-memory EventBus"
 ├── What is an EventBus? (pub/sub within a process)
 ├── What events are published? (AttentionEvent — focus state changes)
 ├── Who subscribes? (Dashboard via WebSocket, SQLite Writer, Analytics Engine)
 ├── Why EventBus vs direct method calls? (decoupling, Open/Closed Principle)
 ├── What ordering guarantees? (FIFO per publisher)
 ├── Is it synchronous or async dispatch?
 ├── What pattern? (Observer pattern)
 └── What would need to change for multi-process instead of multi-thread?
```

### Bullet 3: "Achieved non-blocking inference by isolating analytics from the critical path, applying Shannon entropy to dynamically adapt inference frames and slash CPU consumption by 93%."

```
Claim: "non-blocking inference"
 ├── What is the "critical path"? (camera → inference → state update → dashboard)
 ├── What is NOT on the critical path? (analytics computation, DB writes, report generation)
 ├── How is analytics isolated? (subscribes to EventBus, processes asynchronously)
 ├── What would happen if analytics WERE on the critical path? (inference stalls, frames drop)
 └── What is the latency impact of this isolation?

Claim: "Shannon entropy"
 ├── What is Shannon entropy? (measure of information/uncertainty in a signal)
 ├── How is it computed on a video frame? (pixel intensity histogram → entropy formula)
 ├── What does high entropy mean? (lots of variation = motion = need to run inference)
 ├── What does low entropy mean? (uniform = still = can skip inference)
 ├── What is the formula? H = -Σ p(x) log₂ p(x)
 ├── Why Shannon entropy vs frame differencing?
 ├── Why Shannon entropy vs optical flow?
 ├── What threshold is used? How was it tuned?
 ├── What is the tradeoff? (missing detections when entropy is low but state changes)
 └── How does this relate to adaptive sampling? (sampler.py)

Claim: "dynamically adapt inference frames"
 ├── What does "adapt" mean? (skip YOLO inference when entropy is low)
 ├── How often is YOLO called at baseline? (every frame = 30 FPS)
 ├── How often is YOLO called with adaptation? (only when motion detected)
 ├── What is the effective inference rate?
 └── How does this affect detection accuracy?

Claim: "93% CPU reduction"
 ├── What was the baseline CPU usage?
 ├── What was the optimized CPU usage?
 ├── How was this measured? (benchmarks/ or manual profiling)
 ├── What workload? (sitting still vs active movement)
 ├── Is 93% the average reduction or peak reduction?
 ├── What tradeoff does this introduce? (slower response to sudden phone pickup?)
 ├── How reproducible is this measurement?
 └── What other optimizations contributed? (just entropy or also thread isolation?)
```

### Focus Lock Technology Map
```
Focus Lock
 ├── Python (threading, queue.Queue, dataclasses)
 ├── YOLOv8 (object detection: phone, eating, drinking, talking)
 │    ├── Architecture (anchor-free, CSPDarknet backbone)
 │    ├── Model variants (n/s/m/l/x)
 │    ├── Inference pipeline (preprocess → forward → NMS → postprocess)
 │    └── Training data (custom or pretrained?)
 ├── OpenCV (camera capture, frame processing, MJPEG streaming)
 ├── MediaPipe (head-pose estimation, face mesh landmarks)
 ├── Flask (web server, REST API for dashboard)
 ├── Flask-SocketIO / WebSockets (real-time telemetry push)
 ├── SQLite (session storage, attention events, analytics)
 ├── Chart.js (frontend dashboard charts)
 ├── Tkinter (full-screen lock overlay subprocess)
 ├── pyobjc (macOS Accessibility API for active app detection)
 └── Concepts
      ├── Shannon Entropy (information theory, adaptive sampling)
      ├── Event-Driven Architecture (EventBus, pub/sub)
      ├── Producer-Consumer (queue-based thread pipeline)
      ├── Finite State Machine (FocusFSM: IDLE → FOCUSED → DISTRACTED → BREAK)
      ├── Edge AI (on-device inference, privacy, resource constraints)
      ├── Thread Safety (locks, queues, atomic operations)
      └── Adaptive Sampling (entropy-based frame selection)
```

---

## Experience 1: Cordum (Open Source)

```
Verified public contribution: PR #263 (closed, not merged)
 ├── What did you add? (`govern()` + `CordumGovernanceCallback`, 142 lines across two files)
 ├── How does it attach to LangChain? (`Runnable.with_config()` with a legacy callback fallback)
 ├── What endpoint does it call? (`/api/v1/policy/evaluate` via httpx)
 ├── Which metadata is sent? (topic, tenant, capability, risk tags)
 ├── What happens on DENY / REQUIRE_HUMAN? (`PermissionError` before tool execution)
 ├── Is it a sidecar or async job pipeline? (No; it is a callback-based policy adapter)
 ├── Was it merged? (No; the later merged #264 credited #263 as design inspiration)
 └── What did you learn? (framework compatibility and accurate scope/ownership)
```

## Experience 2: Research Intern

```
Bullet: "Developed 20+ Python/C++ simulation pipelines for ArduSub ROV/AUV testing"
 ├── What is ArduPilot? ArduSub?
 ├── What is an ROV? AUV? Difference?
 ├── What is SITL? HITL?
 ├── What does "simulation pipeline" mean? (sensor input → control logic → vehicle behavior → output)
 ├── Why Python AND C++? (Python for orchestration, C++ for ArduPilot core)
 ├── What counts as "one pipeline"?
 ├── What was the research question?
 └── What was YOUR contribution vs the team's?

Bullet: "Integrated OpenCV-based vision modules into ArduPilot simulation pipelines for underwater feature tracking"
 ├── What features were tracked? (objects? terrain? markers?)
 ├── Why is underwater vision hard? (turbidity, lighting, color absorption, distortion)
 ├── What OpenCV techniques? (template matching? feature detection? optical flow?)
 ├── How does OpenCV integrate with ArduPilot? (MAVLink protocol)
 ├── What is MAVLink?
 └── How do you validate that vision works in simulation vs real underwater?

Bullet: "Validated simulation outputs through iterative experimentation"
 ├── What does "validation" mean here? (sim matches expected behavior)
 ├── What was the validation methodology?
 ├── How many iterations?
 ├── What changed between iterations?
 └── What was the sim-to-real gap?
```

---

## Skills — Full Drill-Down Map

```
Languages
 ├── Python → GIL, asyncio, generators, decorators, context managers, memory management, type hints
 ├── C++ → pointers, smart pointers, RAII, STL, OOP, memory management, templates
 ├── C → pointers, memory allocation, structs, function pointers
 ├── JavaScript → event loop, closures, promises, async/await, prototype chain
 └── SQL → joins, subqueries, window functions, indexing, optimization, CTEs

Backend & Cloud
 ├── FastAPI → async endpoints, dependency injection, Pydantic, middleware, background tasks, OpenAPI
 ├── REST APIs → HTTP methods, status codes, idempotency, versioning, HATEOAS, authentication
 ├── Docker → Dockerfile, layers, caching, multi-stage, compose, networking, volumes
 ├── Kubernetes → Pods, Services, Deployments, ReplicaSets, ConfigMaps, HPA, namespaces
 ├── AWS → EC2, S3, Lambda, RDS, ECS, ECR, IAM, VPC (depth needs confirmation)
 ├── Celery → task queues, workers, brokers, result backends, retries, dead letter queues
 └── RabbitMQ → exchanges, queues, bindings, acknowledgments, durability

Databases
 ├── PostgreSQL → ACID, transactions, indexing (B-tree, GIN, GiST), EXPLAIN, connection pooling, MVCC
 ├── Redis → data structures, Lua scripts, persistence (RDB/AOF), replication, eviction, pub/sub
 ├── SQLite → when to use, WAL mode, limitations, concurrent access
 ├── ChromaDB → vector store, embedding indexing, similarity search, HNSW
 └── pgvector → PostgreSQL extension, vector indexing, cosine/L2 distance

ML & LLM Systems
 ├── RAG → full pipeline, chunking, retrieval, generation, evaluation, hybrid search
 ├── Vector Search → ANN (HNSW, IVF), exact vs approximate, distance metrics
 ├── Information Retrieval → precision, recall, NDCG, MRR, BM25
 ├── LLM Evaluation → faithfulness, relevance, LLM-as-judge, benchmarks
 ├── YOLOv8 → anchor-free detection, architecture, inference pipeline, NMS
 └── OpenCV → image processing, video capture, contour detection, color spaces

Developer Tools
 ├── Git → internals, branching, rebase vs merge, cherry-pick, reflog, bisect
 ├── GitHub Actions → workflows, triggers, runners, artifacts, secrets
 ├── Linux → filesystem, permissions, processes, networking commands, shell scripting
 └── OpenTelemetry → traces, metrics, logs, instrumentation, exporters

Software Engineering
 ├── System Design → load balancing, caching, databases, message queues, consistency
 ├── Distributed Systems → CAP, consensus, replication, partitioning, idempotency
 ├── Concurrency → threads vs processes, race conditions, deadlocks, mutexes, semaphores
 ├── AsyncIO → event loop, coroutines, tasks, gather, when to use vs threads
 ├── Event-Driven Architecture → EventBus, pub/sub, event sourcing, CQRS
 └── OOP → 4 pillars, SOLID, design patterns, composition vs inheritance
```
