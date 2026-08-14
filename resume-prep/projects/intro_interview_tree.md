# Interview Follow-up Tree

> Use this as a drill map, not a script. Before an interview, replace or remove any project-specific implementation detail, tool, metric, or result that you cannot verify from your own code, tests, or notes. A precise, honest answer is stronger than a detailed but unsupported one.

## Topic 1: LLM Cost Guard — RED

### L1 — Basic
**Q:** What is LLM Cost Guard and what problem does it solve?
**Strong answer must contain:** An LLM proxy/gateway that sits between applications and LLM providers to enforce token budgets across multiple providers.
**Depth for internship:** Understands the role of a proxy in a system architecture.
**Common wrong answer:** Describing it as an LLM itself or focusing only on API routing without mentioning budget enforcement.
**Likely follow-up:** How does it track budgets across different providers with varying token costs?

### L2 — Project-Specific
**Q:** You mentioned it acts as a proxy. How does it handle requests across different providers like OpenAI and Anthropic?
**Strong answer must contain:** Normalizes requests using a unified format (like LiteLLM), tracks token counts dynamically, and routes based on policy or availability.
**Depth for internship:** Knows how to design an abstraction layer for different external APIs.
**Common wrong answer:** Saying it just forwards JSON without explaining the payload transformation or cost normalization.
**Likely follow-up:** How did you handle cases where a provider's API changed or went down?

### L3 — Deep Technical
**Q:** How does a proxy architecture introduce latency, and how did you minimize it?
**Strong answer must contain:** Proxy adds network hop overhead. Mitigated using semantic caching, asynchronous I/O (FastAPI), and fast in-memory budget checks (Redis).
**Depth for internship:** Can articulate the trade-offs of adding a middle layer.
**Common wrong answer:** Ignoring the network hop or claiming it adds zero latency.
**Likely follow-up:** Tell me more about the semantic caching implementation.

### L4 — Challenge / Trade-off
**Q:** What's the main challenge of building a unified proxy for APIs that have different rate limits and token calculation methods?
**Strong answer must contain:** The trade-off between normalizing everything (loss of provider-specific features) vs. passing raw requests (harder to track exact costs before the response).
**Depth for internship:** Understands abstraction leaks and cost estimation challenges.
**Common wrong answer:** Claiming you can just count words for all models.
**Likely follow-up:** How do you handle streaming responses where the final token count isn't known upfront?

### L5 — System Design Extension
**Q:** If we deploy LLM Cost Guard globally across multiple regions, how do you handle global budget enforcement?
**Strong answer must contain:** Cross-region replication challenges, eventual consistency vs. strong consistency trade-offs, perhaps using Redis CRDTs or a centralized budget service with local approximate enforcement.
**Depth for internship:** Aware of distributed systems consistency issues.
**Common wrong answer:** Suggesting a single centralized database for all global requests (too much latency).
**Likely follow-up:** How would you design a local "soft budget" that reconciles asynchronously?

## Topic 2: Redis — RED

### L1 — Basic
**Q:** Why did you choose Redis for the budget checks instead of a relational database like PostgreSQL?
**Strong answer must contain:** In-memory speed, single-threaded execution model, and built-in atomic operations.
**Depth for internship:** Knows the difference between in-memory cache and persistent disk storage.
**Common wrong answer:** "It's just faster" without explaining *why* (in-memory, single-threaded).
**Likely follow-up:** What data structures in Redis did you use?

### L2 — Project-Specific
**Q:** What specific Redis data structures did you use to track budgets?
**Strong answer must contain:** Hashes for user/project budgets, maybe Strings/Integers for simple counters, and Lua scripts acting on these keys.
**Depth for internship:** Can map application logic to specific Redis types.
**Common wrong answer:** Saying "tables" or generic JSON blobs without mentioning keys/hashes.
**Likely follow-up:** How did you handle key expiration or resetting budgets (e.g., monthly limits)?

### L3 — Deep Technical
**Q:** Redis is single-threaded. How does it handle high concurrency without becoming a bottleneck?
**Strong answer must contain:** I/O multiplexing (epoll/kqueue) to handle many connections, and fast in-memory operations meaning each command takes microseconds.
**Depth for internship:** Understands the event loop model.
**Common wrong answer:** Believing Redis uses multiple threads for command execution.
**Likely follow-up:** What happens if a Lua script takes too long to execute?

### L4 — Challenge / Trade-off
**Q:** Redis stores data in memory. What happens if the Redis instance crashes? Do you lose all budget data?
**Strong answer must contain:** Redis persistence options (RDB snapshots, AOF logs) and how you balanced performance vs. durability.
**Depth for internship:** Knows about RDB/AOF or that data loss is possible if unconfigured.
**Common wrong answer:** Believing Redis is purely volatile and data is always lost, or assuming it's perfectly durable by default.
**Likely follow-up:** How did you reconcile the Redis cache with the persistent PostgreSQL database?

### L5 — System Design Extension
**Q:** If the traffic grows 100x, how do you scale the Redis component of your system?
**Strong answer must contain:** Redis Cluster for partitioning keys, read replicas if read-heavy (though budget checks are writes), and consistent hashing.
**Depth for internship:** Knows about sharding/partitioning in databases.
**Common wrong answer:** "Just add more RAM to the server."
**Likely follow-up:** How does partitioning affect the atomicity of your Lua scripts?

## Topic 3: Redis Lua Atomicity — RED

### L1 — Basic
**Q:** What is a Lua script in Redis and why did you use it?
**Strong answer must contain:** A way to send custom logic to Redis to be executed server-side. Used it to ensure atomicity of the check-and-decrement operation.
**Depth for internship:** Knows that scripts execute on the database side to save network round trips.
**Common wrong answer:** Thinking Lua is just a configuration language or confusing it with client-side code.
**Likely follow-up:** Why couldn't you just use Redis transactions (MULTI/EXEC)?

### L2 — Project-Specific
**Q:** Walk me through the exact logic inside your Redis Lua script.
**Strong answer must contain:** Retrieve current budget, compare against requested amount, if sufficient decrement and return success, else return failure.
**Depth for internship:** Can write or explain the pseudocode of the script clearly.
**Common wrong answer:** Explaining it as multiple separate Redis calls instead of one script block.
**Likely follow-up:** How did you test the Lua script?

### L3 — Deep Technical
**Q:** Why does using a Lua script guarantee atomicity in Redis?
**Strong answer must contain:** Redis is single-threaded for command execution. When a Lua script runs, it blocks all other commands until it completes.
**Depth for internship:** Links single-threaded nature to atomicity.
**Common wrong answer:** Saying Redis uses locks/mutexes internally for scripts.
**Likely follow-up:** What is the performance impact of blocking all other commands?

### L4 — Challenge / Trade-off
**Q:** What are the drawbacks of using Lua scripts in Redis?
**Strong answer must contain:** Blocking other operations (so scripts must be fast), harder to debug/version control, and issues with Redis Cluster if keys hash to different nodes.
**Depth for internship:** Understands that "blocking everything" is dangerous if the script is slow.
**Common wrong answer:** Believing there are no drawbacks to server-side execution.
**Likely follow-up:** Did you encounter any issues with keys not being on the same shard in a cluster setup?

### L5 — System Design Extension
**Q:** If your Lua script needs to check a budget key and update a separate log key, how do you handle this in a Redis Cluster where they might live on different nodes?
**Strong answer must contain:** Hashtags (e.g., `{user123}:budget` and `{user123}:log`) to force keys onto the same hash slot, allowing the script to run safely.
**Depth for internship:** Knows about Redis hash slots and key tagging.
**Common wrong answer:** Assuming Redis Cluster automatically handles cross-shard scripts.
**Likely follow-up:** What if the log needs to be aggregated globally across all users?

## Topic 4: Concurrent Budget Enforcement — RED

### L1 — Basic
**Q:** What was the core technical problem with concurrent budget over-authorization in your proxy?
**Strong answer must contain:** Time-Of-Check to Time-Of-Use (TOCTOU) race condition. Two requests check the budget simultaneously, both see enough funds, and both subtract, leading to a negative balance.
**Depth for internship:** Can explain a race condition clearly with an example.
**Common wrong answer:** Describing a general bug rather than a concurrency-specific race condition.
**Likely follow-up:** How did you identify this issue was happening?

### L2 — Project-Specific
**Q:** How exactly did your Lua script fix the TOCTOU issue?
**Strong answer must contain:** By combining the read (check) and write (decrement) into a single, atomic server-side operation, preventing interleaving.
**Depth for internship:** Connects atomicity directly to preventing the race condition.
**Common wrong answer:** Saying you added a lock in Python.
**Likely follow-up:** Why not just use a lock in your Python code?

### L3 — Deep Technical
**Q:** Why is a distributed lock (e.g., Redlock) a worse solution here than an atomic Lua script?
**Strong answer must contain:** Locks add latency (extra network round trips to acquire/release), risk of deadlocks, and lock contention under high load. Lua script avoids lock overhead entirely.
**Depth for internship:** Understands lock overhead vs. atomic operations.
**Common wrong answer:** Believing distributed locks are always the best solution for concurrency.
**Likely follow-up:** When *would* you use a distributed lock instead?

### L4 — Challenge / Trade-off
**Q:** What happens if the LLM request fails after you've already decremented the budget atomically?
**Strong answer must contain:** Need a rollback mechanism or compensation transaction to refund the tokens if the API call fails.
**Depth for internship:** Aware of partial failures and compensation in distributed systems.
**Common wrong answer:** Ignoring the failure case or assuming the LLM API always succeeds.
**Likely follow-up:** How did you implement this refund mechanism?

### L5 — System Design Extension
**Q:** How would you solve this if you were using PostgreSQL instead of Redis for the budget?
**Strong answer must contain:** `SELECT ... FOR UPDATE` (row-level locking) or optimistic concurrency control using a version column or `UPDATE balance = balance - amount WHERE balance >= amount`.
**Depth for internship:** Knows SQL concurrency control mechanisms.
**Common wrong answer:** Just running standard SELECT and UPDATE in a regular transaction (which still allows race conditions depending on isolation level).
**Likely follow-up:** What isolation level would be required?

## Topic 5: Locust / Load Testing — YELLOW

### L1 — Basic
**Q:** What is Locust and why did you use it over other tools like JMeter?
**Strong answer must contain:** A Python-based load testing tool. Used it to simulate high concurrency to validate that the race condition was fixed. Python makes it easy to write complex test scenarios.
**Depth for internship:** Knows what load testing is and why to use it.
**Common wrong answer:** Confusing load testing with unit testing.
**Likely follow-up:** How many concurrent users did you simulate?

### L2 — Project-Specific
**Q:** You said you had 'zero budget overruns' during the Locust test. How exactly did you measure that?
**Strong answer must contain:** Set a specific budget, fired X concurrent requests that sum to > budget, and asserted that exactly the right number succeeded and the rest were rejected, with the final Redis balance exactly zero (not negative).
**Depth for internship:** Can design a valid test for a race condition.
**Common wrong answer:** "I just ran it and it didn't crash."
**Likely follow-up:** Did you run this test locally or against a deployed environment?

### L3 — Deep Technical
**Q:** When load testing from a single machine, what are the common bottlenecks that might skew your results?
**Strong answer must contain:** CPU limits on the generator, ephemeral port exhaustion (TCP connections), or network bandwidth limits.
**Depth for internship:** Aware of client-side limitations in load testing.
**Common wrong answer:** Assuming the server is always the bottleneck.
**Likely follow-up:** How do you run distributed Locust tests?

### L4 — Challenge / Trade-off
**Q:** Did you have to mock the actual LLM API during the load test? Why?
**Strong answer must contain:** Yes, to avoid spending real money, avoid hitting provider rate limits, and isolate the test to just the proxy's concurrency logic.
**Depth for internship:** Understands isolating components for testing.
**Common wrong answer:** Saying they hit the real OpenAI API thousands of times.
**Likely follow-up:** How did you simulate the latency of the LLM API in your mock?

### L5 — System Design Extension
**Q:** If we want to continuously load test this proxy in a CI/CD pipeline, how would you design that?
**Strong answer must contain:** Spinning up an ephemeral environment, running a short Locust headless test on every PR, breaking the build if error rate > 0% or latency > threshold.
**Depth for internship:** Knows how to integrate performance testing into CI.
**Common wrong answer:** Running a massive DDOS test on every commit.
**Likely follow-up:** How do you prevent the load test from affecting production if running against staging?

## Topic 6: RAGOS — RED

### L1 — Basic
**Q:** What is RAGOS and what problem does it solve?
**Strong answer must contain:** RAGOS is an AutoML-like tool for RAG pipelines. It automates the search for the best configuration (chunk size, retriever type, model) given latency and cost constraints.
**Depth for internship:** Can clearly summarize the value prop of optimizing RAG.
**Common wrong answer:** Describing it just as a generic RAG app rather than an optimization tool.
**Likely follow-up:** What were the main parameters you were optimizing?

### L2 — Project-Specific
**Q:** How do you define a "configuration" in RAGOS?
**Strong answer must contain:** A combination of parameters: chunk size, overlap, embedding model, vector DB index type, top-k retrieval, and the final generation LLM.
**Depth for internship:** Knows the key hyperparameters of a RAG pipeline.
**Common wrong answer:** Only mentioning the LLM.
**Likely follow-up:** How did you evaluate the quality of a specific configuration?

### L3 — Deep Technical
**Q:** How did you build the plugin architecture (BaseRetriever/BaseEmbedder)?
**Strong answer must contain:** Abstract base classes in Python defining a standard interface, allowing the optimizer to swap components transparently.
**Depth for internship:** Understands interface design and polymorphism.
**Common wrong answer:** Hardcoding all combinations in a massive if/else block.
**Likely follow-up:** How do you handle a plugin that fails mid-trial?

### L4 — Challenge / Trade-off
**Q:** Evaluating RAG pipelines is notoriously hard. How did you measure "quality" automatically?
**Strong answer must contain:** LLM-as-a-judge (e.g., using GPT-4 to evaluate context relevance, faithfulness, and answer relevance).
**Depth for internship:** Aware of current RAG evaluation techniques (like RAGAS/TruLens concepts).
**Common wrong answer:** Using simple BLEU/ROUGE scores which don't work well for semantics.
**Likely follow-up:** How do you trust the LLM judge, and how much did evaluation cost?

### L5 — System Design Extension
**Q:** If RAGOS needs to evaluate 10,000 configurations, how would you distribute this workload?
**Strong answer must contain:** A message queue (RabbitMQ/Celery) distributing trials to a pool of worker nodes, writing results to a centralized DB (PostgreSQL).
**Depth for internship:** Can design a basic distributed task queue system.
**Common wrong answer:** Running it all in a massive multithreaded Python script on one machine.
**Likely follow-up:** How do you handle workers crashing mid-evaluation?

## Topic 7: Random Search — YELLOW

### L1 — Basic
**Q:** Why did you choose Random Search over Grid Search?
**Strong answer must contain:** Grid search is too expensive/slow as the dimension of parameters grows (curse of dimensionality). Random search explores the space more efficiently.
**Depth for internship:** Knows basic optimization strategies.
**Common wrong answer:** "Random search is just easier to code."
**Likely follow-up:** Why not Bayesian Optimization?

### L2 — Project-Specific
**Q:** How did you ensure your Random Search didn't just pick terrible configurations most of the time?
**Strong answer must contain:** Constraint filtering before the trial—only sampling from distributions that were likely to meet the cost/latency bounds.
**Depth for internship:** Connects the search strategy to their specific implementation.
**Common wrong answer:** "I just let it run for a long time."
**Likely follow-up:** How do you sample randomly from categorical variables (like models)?

### L3 — Deep Technical
**Q:** Explain the mathematical intuition behind why Random Search often outperforms Grid Search in high dimensions.
**Strong answer must contain:** In high dimensions, usually only a few hyperparameters actually matter. Random search tests a unique value for the important parameters on every trial, while grid search wastes time testing the same values repeatedly across unimportant dimensions.
**Depth for internship:** Understands the concept of low effective dimensionality.
**Common wrong answer:** Believing it's purely about saving compute time, rather than better coverage of important dimensions.
**Likely follow-up:** How do you decide when to stop the Random Search?

### L4 — Challenge / Trade-off
**Q:** What is the main drawback of Random Search in this context?
**Strong answer must contain:** It doesn't learn from previous iterations (unlike Bayesian optimization). It might repeatedly test areas of the space that yield poor results.
**Depth for internship:** Knows the limitations of memoryless search algorithms.
**Common wrong answer:** Saying it's too slow (it's faster than grid).
**Likely follow-up:** If you were to upgrade to Bayesian Optimization, what library would you use?

### L5 — System Design Extension
**Q:** How do you parallelize Random Search?
**Strong answer must contain:** It's embarrassingly parallel since trials are independent. Just dispatch N random configs to N workers simultaneously.
**Depth for internship:** Understands the difference between parallelizable and sequential algorithms.
**Common wrong answer:** Trying to coordinate state between workers (unnecessary).
**Likely follow-up:** How is parallelizing Random Search easier than parallelizing Bayesian Optimization?

## Topic 8: Pareto Frontier / Multi-Objective Optimization — YELLOW

### L1 — Basic
**Q:** What is a Pareto frontier in the context of RAGOS?
**Strong answer must contain:** The set of configurations where you cannot improve one metric (e.g., accuracy) without degrading another (e.g., cost or latency).
**Depth for internship:** Can define Pareto efficiency simply.
**Common wrong answer:** Thinking it's just a single "best" overall configuration.
**Likely follow-up:** Why is returning a frontier better than returning one single result?

### L2 — Project-Specific
**Q:** How did you actually compute the Pareto frontier from the trial results?
**Strong answer must contain:** Iterating through results and discarding any configuration that is strictly "dominated" (worse on all axes) by another configuration.
**Depth for internship:** Can describe the algorithmic step of finding non-dominated sorting.
**Common wrong answer:** Just sorting by one metric.
**Likely follow-up:** What is the time complexity of a naive Pareto calculation?

### L3 — Deep Technical
**Q:** Why not just use a weighted sum (e.g., `Score = 0.5*Accuracy - 0.3*Latency - 0.2*Cost`) to find the best model?
**Strong answer must contain:** Weighted sums require choosing arbitrary weights upfront, which users often can't do accurately. They also fail to find solutions on concave parts of the Pareto front.
**Depth for internship:** Understands the mathematical limitations of scalarization.
**Common wrong answer:** "It's too hard to code a weighted sum."
**Likely follow-up:** How does a user interact with the Pareto frontier you return?

### L4 — Challenge / Trade-off
**Q:** What happens when you have many dimensions (accuracy, latency, cost, memory, etc.)? Does the Pareto concept still work well?
**Strong answer must contain:** "Pareto explosion" – in high dimensions, almost every point becomes non-dominated, making the frontier uselessly large.
**Depth for internship:** Aware of the curse of dimensionality in multi-objective problems.
**Common wrong answer:** Assuming it scales perfectly to 10+ metrics.
**Likely follow-up:** How do you mitigate Pareto explosion?

### L5 — System Design Extension
**Q:** If we have millions of data points streamed continuously, how do you maintain a rolling Pareto frontier efficiently?
**Strong answer must contain:** Using specialized data structures like quadtrees or maintaining a running set of non-dominated points, checking new points against the set and pruning efficiently.
**Depth for internship:** Thinks about algorithmic efficiency for streaming data.
**Common wrong answer:** Recomputing the entire frontier from scratch on every new data point.
**Likely follow-up:** What DB index might help with this?

## Topic 9: Latency and Cost Constraints in RAGOS — YELLOW

### L1 — Basic
**Q:** Your intro mentions "constraint filtering before each trial". How does that work?
**Strong answer must contain:** Calculating the theoretical max cost or expected latency of a configuration before actually running the API call, and skipping it if it violates user bounds.
**Depth for internship:** Understands proactive filtering to save resources.
**Common wrong answer:** Filtering *after* the API call (which wastes money).
**Likely follow-up:** How do you predict latency before making the call?

### L2 — Project-Specific
**Q:** How do you predict the cost of a configuration without running it?
**Strong answer must contain:** Using fixed formulas: cost = (number of documents * chunk size * embedding cost) + (top-k * chunk size + prompt size * LLM cost).
**Depth for internship:** Can break down the mathematical cost model of RAG.
**Common wrong answer:** Guessing or saying it's impossible.
**Likely follow-up:** What about output tokens, which you can't predict?

### L3 — Deep Technical
**Q:** Latency is highly variable. How can you reliably filter based on latency constraints upfront?
**Strong answer must contain:** You can't perfectly predict it, but you can use historical baselines, simple heuristics (e.g., GPT-4 is generally slower than Haiku), or calculate a lower bound (e.g., minimum network TTFB).
**Depth for internship:** Acknowledges the non-deterministic nature of network latency.
**Common wrong answer:** Claiming you had a perfect latency prediction model.
**Likely follow-up:** How did you handle a config that passed the filter but violated the constraint during the real run?

### L4 — Challenge / Trade-off
**Q:** What is the risk of filtering too aggressively before the trial?
**Strong answer must contain:** You might prune potentially great configurations just because your heuristic cost/latency model slightly overestimated them.
**Depth for internship:** Understands false positives in filtering logic.
**Common wrong answer:** Believing strict filters are always better.
**Likely follow-up:** Should the filters be soft or hard bounds?

### L5 — System Design Extension
**Q:** If API pricing changes frequently, how do you keep your cost constraints accurate without hardcoding prices?
**Strong answer must contain:** Integrating with an external pricing registry (like LiteLLM's model cost map) and fetching pricing data dynamically at startup or via a cron job.
**Depth for internship:** Knows how to design dynamic configuration management.
**Common wrong answer:** Hardcoding all prices in a JSON file forever.
**Likely follow-up:** How do you handle caching this pricing data?

## Topic 10: Focus Lock — RED

### L1 — Basic
**Q:** What is Focus Lock and what does it do?
**Strong answer must contain:** A local computer vision app using YOLOv8 and MediaPipe to detect user focus/distraction during desk sessions.
**Depth for internship:** Clear elevator pitch of the project.
**Common wrong answer:** Focusing too much on the ML models without explaining the user experience.
**Likely follow-up:** Why did you use both YOLO and MediaPipe?

### L2 — Project-Specific
**Q:** Walk me through the architecture of the 4-thread model you used.
**Strong answer must contain:** Camera thread (captures frames), Inference thread (runs models), WebSocket thread (sends data to UI), and Database thread (logs states).
**Depth for internship:** Understands why UI, I/O, and heavy compute must be separated.
**Common wrong answer:** Putting a `while True` loop with `time.sleep` all in one file.
**Likely follow-up:** How did you communicate between these threads safely?

### L3 — Deep Technical
**Q:** How did you handle the Python GIL (Global Interpreter Lock) when doing heavy inference?
**Strong answer must contain:** OpenCV/MediaPipe/YOLO run heavily in C++ extensions, which release the GIL during execution, allowing Python threads to run concurrently during the actual compute.
**Depth for internship:** Knows that C extensions can bypass GIL limitations.
**Common wrong answer:** Saying "threads in Python are truly parallel" without understanding the GIL.
**Likely follow-up:** Why use threading instead of multiprocessing?

### L4 — Challenge / Trade-off
**Q:** Managing state transitions (FocusFSM) can be buggy (e.g., flickering between FOCUSED and DISTRACTED). How did you stabilize it?
**Strong answer must contain:** Debouncing, temporal smoothing, or requiring a state to be observed for N consecutive frames before transitioning.
**Depth for internship:** Understands hysteresis in state machines based on noisy sensor data.
**Common wrong answer:** Trusting every single frame individually.
**Likely follow-up:** What happens if the person just leaves the room?

### L5 — System Design Extension
**Q:** If we wanted to make Focus Lock a SaaS application tracking focus across 10,000 remote employees, what changes?
**Strong answer must contain:** Privacy concerns (process video locally, only send metadata/states to server), building a scalable backend (WebSocket gateways, time-series DB for analytics).
**Depth for internship:** Recognizes privacy implications of video and scales the data layer.
**Common wrong answer:** Streaming raw 1080p video from 10k users to a central server (massive bandwidth/privacy issue).
**Likely follow-up:** How do you handle intermittent network connectivity for the local client?

## Topic 11: YOLOv8 — YELLOW

### L1 — Basic
**Q:** What is YOLOv8 and what specific objects were you detecting?
**Strong answer must contain:** You Only Look Once. It's a real-time object detection model. Used it to detect phones or other distractions.
**Depth for internship:** Knows what object detection is vs. image classification.
**Common wrong answer:** Saying it classifies the whole image as "distracted".
**Likely follow-up:** Did you train it from scratch or use pre-trained weights?

### L2 — Project-Specific
**Q:** Your resume mentions YOLOv8 is an "anchor-free" detector. What does that mean?
**Strong answer must contain:** It predicts bounding box centers and sizes directly rather than relying on predefined anchor box ratios, making it more flexible and faster to train.
**Depth for internship:** Basic knowledge of modern object detection architecture.
**Common wrong answer:** Not knowing what an anchor box is.
**Likely follow-up:** What is Non-Maximum Suppression (NMS) and why is it needed?

### L3 — Deep Technical
**Q:** YOLO is heavy. How did you get it to run in <30ms locally?
**Strong answer must contain:** Using the nano model (YOLOv8n), running inference on ONNX runtime or TensorRT, and resizing input frames appropriately.
**Depth for internship:** Knows about model quantization/formats and size trade-offs.
**Common wrong answer:** "It's just naturally fast."
**Likely follow-up:** Did you run this on CPU or GPU?

### L4 — Challenge / Trade-off
**Q:** YOLO struggles with small objects or occlusions. How did you handle a partially hidden phone?
**Strong answer must contain:** Either fine-tuning on a custom dataset, lowering the confidence threshold, or relying on temporal consistency (if it was a phone a second ago, it still is).
**Depth for internship:** Aware of edge cases in computer vision in the wild.
**Common wrong answer:** Assuming the model is perfectly accurate.
**Likely follow-up:** Tell me about the custom dataset you mentioned (97.2% recall).

### L5 — System Design Extension
**Q:** If you wanted to deploy this model to a low-power edge device like a Raspberry Pi, what optimization techniques would you apply?
**Strong answer must contain:** INT8 Quantization, pruning, using TFLite, or swapping to an even lighter architecture.
**Depth for internship:** Knows edge AI optimization concepts.
**Common wrong answer:** Running the raw PyTorch model on a Pi.
**Likely follow-up:** What is the trade-off of INT8 quantization?

## Topic 12: MediaPipe — GREEN

### L1 — Basic
**Q:** What did you use MediaPipe for specifically?
**Strong answer must contain:** Face mesh / head pose estimation and gaze tracking to see if the user is looking at the screen.
**Depth for internship:** Knows the specific API used within MediaPipe.
**Common wrong answer:** Mixing it up with YOLO's object detection.
**Likely follow-up:** Why MediaPipe instead of Dlib or OpenCV Haar Cascades?

### L2 — Project-Specific
**Q:** How do you calculate "gaze" or head pose from a 2D image?
**Strong answer must contain:** MediaPipe provides 3D landmarks. You use the relative positions of the eyes, nose, and chin to estimate a pitch/yaw/roll vector.
**Depth for internship:** Understands basic facial geometry concepts.
**Common wrong answer:** "MediaPipe just returns a boolean."
**Likely follow-up:** What happens if the user wears glasses?

### L3 — Deep Technical
**Q:** MediaPipe operates on a tracking paradigm rather than detection per-frame. How does that improve performance?
**Strong answer must contain:** It runs a heavy face detector once, then uses a lightweight landmark tracker on subsequent frames based on the previous position, saving massive CPU cycles.
**Depth for internship:** Understands detection vs. tracking pipelines.
**Common wrong answer:** Believing it does full detection every single frame.
**Likely follow-up:** When does the tracker fail and force a re-detection?

### L4 — Challenge / Trade-off
**Q:** What happens when the user turns their head completely sideways (profile view)?
**Strong answer must contain:** The tracker loses landmarks and fails. The system must degrade gracefully (e.g., assume they are distracted, or trigger a re-detection).
**Depth for internship:** Aware of physical limitations of 2D face tracking.
**Common wrong answer:** Assuming it tracks perfectly 360 degrees.
**Likely follow-up:** How did you handle lighting changes?

### L5 — System Design Extension
**Q:** If you had to process video streams from 100 cameras centrally to extract head pose, how would you architect the pipeline?
**Strong answer must contain:** Decode frames in hardware, batch frames, run inference on GPUs using TensorRT, not MediaPipe (which is CPU/mobile optimized).
**Depth for internship:** Knows that client-side tools don't always scale to server-side batch processing.
**Common wrong answer:** Running 100 MediaPipe instances on a CPU server.
**Likely follow-up:** What video protocol would you use to stream the data?

## Topic 13: Shannon Entropy — RED

### L1 — Basic
**Q:** What is Shannon Entropy in the context of an image?
**Strong answer must contain:** A measure of information or unpredictability. In an image, high entropy means lots of detail/edges/noise, low entropy means uniform flat color.
**Depth for internship:** Can explain the concept of entropy simply.
**Common wrong answer:** Mixing it up with cross-entropy loss in ML.
**Likely follow-up:** How do you calculate it mathematically?

### L2 — Project-Specific
**Q:** Why did you apply entropy to *frame differences* rather than the raw frames themselves?
**Strong answer must contain:** A raw frame always has high entropy (background, desk, etc.). A difference image isolates *motion*. If nothing moves, the diff is black (entropy ≈ 0). If someone moves, the diff has high entropy.
**Depth for internship:** Clever application of basic math to isolate motion.
**Common wrong answer:** Misunderstanding what a frame difference is.
**Likely follow-up:** How do you compute the frame difference? (Absolute difference of pixel values).

### L3 — Deep Technical
**Q:** Walk me through the exact calculation pipeline: from two frames to an entropy float value.
**Strong answer must contain:** Convert to grayscale -> compute absolute difference -> compute histogram (pixel value probabilities) -> apply `H = -sum(p * log(p))`.
**Depth for internship:** Knows the mathematical steps.
**Common wrong answer:** Forgetting the histogram/probability step.
**Likely follow-up:** Why convert to grayscale first?

### L4 — Challenge / Trade-off
**Q:** What happens if the lighting in the room flickers or changes suddenly?
**Strong answer must contain:** The frame difference will show massive changes everywhere, causing high entropy and a false positive for motion.
**Depth for internship:** Recognizes environmental fragility of pixel-based techniques.
**Common wrong answer:** Believing entropy is immune to lighting.
**Likely follow-up:** How would you mitigate lighting flicker?

### L5 — System Design Extension
**Q:** If you wanted to run this entropy calculation on 4K video at 60FPS, how would you optimize it?
**Strong answer must contain:** Downsample the image heavily first (you don't need 4K for motion detection), or run the calculation on the GPU using custom CUDA kernels/OpenCV optimizations.
**Depth for internship:** Understands data reduction for performance.
**Common wrong answer:** Running python `for` loops over 8 million pixels.
**Likely follow-up:** How small can you downsample before you lose motion accuracy?

## Topic 14: Adaptive Sampling — YELLOW

### L1 — Basic
**Q:** How did the entropy threshold decide when to skip YOLO inference?
**Strong answer must contain:** If the entropy of the frame difference is below a threshold (no motion), assume the user is stationary and skip the heavy YOLO/MediaPipe compute.
**Depth for internship:** Connects motion detection directly to compute saving.
**Common wrong answer:** Saying YOLO itself decides when to run.
**Likely follow-up:** How did you determine the exact threshold value?

### L2 — Project-Specific
**Q:** What is the "frozen-phone" edge case you mentioned?
**Strong answer must contain:** If the user is holding a phone completely still, entropy is low (no motion). If you skip YOLO, you might miss the fact that they are currently distracted.
**Depth for internship:** Identifies the logical flaw in purely motion-based triggers.
**Common wrong answer:** Not knowing what the edge case means.
**Likely follow-up:** How did you fix this edge case?

### L3 — Deep Technical
**Q:** How do you balance the trade-off between missing a fast movement (false negative) and running YOLO too often (wasting CPU)?
**Strong answer must contain:** Tuning the threshold via empirical testing (ROC curve), or using a dynamic threshold based on recent activity levels.
**Depth for internship:** Understands tuning thresholds for precision/recall tradeoffs.
**Common wrong answer:** Assuming there is a single perfect magical threshold.
**Likely follow-up:** What is a "liveness check" in this context?

### L4 — Challenge / Trade-off
**Q:** If the user leaves the frame completely, the frame difference entropy drops to zero. How does the system know they are gone if YOLO is skipped?
**Strong answer must contain:** Need a periodic forced inference (e.g., run YOLO every N frames regardless of entropy) to validate absolute state.
**Depth for internship:** Understands the necessity of periodic keyframes (like I-frames in video compression).
**Common wrong answer:** The system just assumes they are still there forever.
**Likely follow-up:** How often did you force a check?

### L5 — System Design Extension
**Q:** If you were designing a smart security camera, how would you implement adaptive sampling at the hardware level?
**Strong answer must contain:** A low-power PIR (passive infrared) motion sensor wakes up the main SOC/Camera only when physical heat motion is detected.
**Depth for internship:** Knows hardware analogies to software concepts.
**Common wrong answer:** Running a neural network 24/7 on a battery camera.
**Likely follow-up:** What is the software equivalent of a PIR sensor here?

## Topic 15: CPU / Inference Optimization — YELLOW

### L1 — Basic
**Q:** You claimed a 93% reduction in YOLO compute. How exactly was this measured?
**Strong answer must contain:** Counted the number of total frames vs. the number of frames YOLO was actually executed on during a typical "stationary desk session".
**Depth for internship:** Knows how to define a metric explicitly.
**Common wrong answer:** Guessing the number.
**Likely follow-up:** What does a "stationary desk session" entail?

### L2 — Project-Specific
**Q:** How did the interaction between Python's GIL and C++ inference help your application's responsiveness?
**Strong answer must contain:** Because YOLO/OpenCV release the GIL, the Python threads handling the UI/WebSocket could respond instantly without being blocked by the heavy 30ms inference step.
**Depth for internship:** Deep understanding of async/threading in Python.
**Common wrong answer:** Saying the GIL makes it faster (the GIL makes things slower; releasing it helps).
**Likely follow-up:** How did you profile the CPU usage?

### L3 — Deep Technical
**Q:** What tools would you use in Python to prove that your threads are running concurrently and not blocking each other?
**Strong answer must contain:** Profilers like cProfile, PySpy, or logging timestamps before and after thread execution blocks to visualize overlap.
**Depth for internship:** Knows profiling tools.
**Common wrong answer:** "I just looked at Task Manager."
**Likely follow-up:** What is thread contention?

### L4 — Challenge / Trade-off
**Q:** What is the overhead of thread context switching in your 4-thread model?
**Strong answer must contain:** Context switching takes CPU cycles. If threads are switching too fast (e.g., for trivial tasks), the overhead outweighs the parallelism benefits.
**Depth for internship:** Understands context switching penalties.
**Common wrong answer:** Believing more threads always equals more speed.
**Likely follow-up:** Could this have been done with asyncio instead of threading?

### L5 — System Design Extension
**Q:** If you ported this entire application to C++, what performance gains would you expect and why?
**Strong answer must contain:** Elimination of the Python interpreter overhead, no GIL to worry about, better memory management, but at the cost of much slower development time.
**Depth for internship:** Can articulate Python vs C++ trade-offs accurately.
**Common wrong answer:** "It would be 1000x faster" (The ML inference is already in C++, so the gain is only on the glue code).
**Likely follow-up:** Would you use raw threads or a library like Boost.Asio?

---
## Resume Technologies Not in Introduction — Interviewer Could Jump To

| Technology | Where Used | Risk Level | What They'd Ask |
|-----------|-----------|------------|----------------|
| **RabbitMQ / Celery** | LLM Cost Guard (Async reconciliation) | HIGH | How does Celery handle task failures? What's the difference between a message broker and a task queue? |
| **FastAPI** | LLM Cost Guard | MED | How does FastAPI handle async requests compared to Flask? What is an ASGI server? |
| **PostgreSQL** | LLM Cost Guard, RAGOS | HIGH | ACID properties, indexing strategies, handling concurrent writes, JOINs vs Denormalization. |
| **Circuit Breaker** | LLM Cost Guard | HIGH | How do you implement a circuit breaker? What are the states (Open, Closed, Half-Open)? |
| **Semantic Caching** | LLM Cost Guard | HIGH | How do you calculate semantic similarity? What threshold did you use? (e.g. cosine similarity). |
| **LiteLLM** | RAGOS | LOW | What is it? (Just an API wrapper). How did it help? |
| **ChromaDB** | RAGOS | MED | How do vector databases work? What indexing algorithm (HNSW)? |
| **ArduPilot / ArduSub** | Research (SITL Simulation) | MED | What is SITL (Software In The Loop)? How do you simulate physics for underwater vehicles? |
| **OpenCV** | Research | MED | Feature tracking algorithms (SIFT, SURF, ORB). How did you handle underwater distortion? |
| **FocusFSM** | Focus Lock | LOW | What is a Finite State Machine? How do you prevent rapid toggling? |
| **Flask-WebSocket** | Focus Lock | MED | How do WebSockets differ from HTTP REST? Why use them here? |
| **SQLite** | Focus Lock | LOW | Why SQLite instead of Postgres here? (Local, embedded, serverless). |
| **Cordum** | Cordum Sidecar | HIGH | What is the Sidecar pattern? How did you refactor sync to async pipelines? |
| **Integration Tests** | Cordum | MED | Difference between unit and integration tests? How did you mock external services? |
