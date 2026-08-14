# LLM Cost Guard - Interview Interrogation Document

## SECTION 1: BULLET INTERROGATION TREES

### Bullet 1: "Implemented atomic Redis token reservation to eliminate double charging under concurrent requests, validated with 7,100+ concurrent integration tests."

## Q. Talk to me about this atomic Redis token reservation. Why was atomicity a problem in the first place?

**Answer:** When multiple concurrent requests try to deduct from the same token budget, we hit a Time-of-Check to Time-of-Use (TOCTOU) race condition. If two requests check the balance simultaneously, they might both see enough tokens, proceed, and then both deduct, causing the budget to go negative. Atomicity ensures the check and deduction happen as a single indivisible operation.

### Follow-up: Walk me through the exact race condition if you didn't have atomicity.

**Answer:** Request A reads a budget of 1000 tokens. Request B simultaneously reads the budget of 1000 tokens. Request A needs 600 tokens, sees it's valid, proceeds, and sets the new balance to 400. Request B needs 800 tokens, also saw the balance as 1000, proceeds, and sets the balance to 200. We just authorized 1400 tokens against a 1000 token budget.

### Follow-up: How did you solve this? Why Redis Lua scripts over WATCH/MULTI/EXEC?

**Answer:** I used a Redis Lua script. While WATCH/MULTI/EXEC provides optimistic locking, it fails under high contention. If 50 requests hit the same budget, 49 will fail their transaction and have to retry, causing a latency spike and CPU churn. A Lua script executes atomically inside Redis's single thread. It evaluates the condition and updates the value in one step, without retries, meaning O(1) latency even under contention.

### Follow-up: Walk me through the Lua script step by step. What is it actually doing?

**Answer:** The script takes the budget key and the requested tokens as arguments. 
1. It reads the current value of the key (or defaults to the maximum limit if it doesn't exist).
2. It compares the current value to the requested tokens.
3. If the value is less than the requested tokens, it returns a 429 (Rate Limited) or insufficient funds error.
4. If sufficient, it deducts the tokens, updates the key's TTL (if it's a rolling window), and returns success along with the new balance.

### Follow-up: What Redis data structures are you using for the rate limiting?

**Answer:** I used simple strings with integer values for absolute budget counters. For time-windowed rate limits (e.g., tokens per minute), I used the generic cell rate algorithm (GCRA) or a sliding window log, but for budget enforcement, it was a decrementing counter on a string key with an expiry matching the reset window.

### Follow-up: What happens if the LLM call fails after you've already reserved the tokens?

**Answer:** This introduces the Reserve-and-Reconcile lifecycle. The Lua script does a *reservation* based on an estimated token count (since we don't know the exact output size yet). If the LLM call fails, the router catches the exception and issues a background task to *release* (refund) those reserved tokens back to the Redis budget.

### Follow-up: What if the LLM call succeeds, but the actual token usage is different from the estimate?

**Answer:** We reconcile. After the LLM returns its usage metadata, we calculate `actual_tokens - estimated_tokens`. If the actual was lower, we refund the difference. If higher, we deduct the difference. This happens asynchronously via Celery so it doesn't block the user's response.

### Follow-up: What if the Celery worker processing the reconciliation crashes before completing the refund?

**Answer:** This risks a leaked reservation (tokens permanently lost). To mitigate this, we need message acknowledgments in RabbitMQ. The message is only ACKed after the Redis update succeeds. If the worker crashes, RabbitMQ re-queues the message to another worker. If the refund fails repeatedly, it goes to a Dead Letter Queue (DLQ) for manual intervention.

### Follow-up: What if Redis crashes mid-Lua execution?

**Answer:** Redis Lua scripts are atomic; they either fully execute or don't. However, if the Redis server itself crashes during execution, the command is lost. Depending on persistence settings (AOF fsync=everysec), we might lose up to 1 second of data. The system would fail-open or fail-closed based on configuration, but we wouldn't end up in a corrupted partial state.

### Follow-up: How do you handle multi-window budgets? For example, 1000 tokens/min AND 10,000 tokens/day?

**Answer:** The Lua script is expanded to check and decrement multiple keys simultaneously. It reads the balance for the minute key and the day key. If *either* check fails, it rejects the request. If *both* pass, it decrements both. This maintains atomicity across multiple time boundaries.

---

### Bullet 2: "Built a production AI Gateway for centralized policy enforcement, intelligent provider routing, and token budget management across multiple LLM providers."

## Q. What does "intelligent provider routing" actually mean in your implementation?

**Answer:** It means the router dynamically selects the underlying LLM API (OpenAI, Anthropic, Groq) based on runtime conditions rather than hardcoded client logic. If a user requests a generic "fast-completion", the router checks the configuration and sends it to Groq. If it's complex reasoning, it routes to GPT-4. It also handles fallbacks: if OpenAI returns a 429, the router automatically retries against an Anthropic equivalent.

### Follow-up: How does the router know which models are equivalent?

**Answer:** I defined a mapping configuration in `config.py` that groups models into capability tiers (e.g., `tier1_reasoning`: gpt-4, claude-3-opus; `tier3_fast`: gpt-3.5, mixtral-groq). The routing logic uses this taxonomy to find substitutes if the primary model fails.

### Follow-up: Where is the policy enforcement happening in the request lifecycle?

**Answer:** It happens in `policy.py`, which is invoked by a FastAPI dependency *before* the request reaches the router or rate limiter. It evaluates the authenticated user's organization, their role, and the requested endpoint against rules stored in PostgreSQL (e.g., "Interns cannot use GPT-4").

### Follow-up: If policies are in PostgreSQL, doesn't querying the DB for every request add massive latency?

**Answer:** Yes, it would. To fix this, I implemented an in-memory cache with a short TTL (e.g., 5 minutes) or a Redis-backed cache for policies. When a request comes in, we check the fast cache first. 

### Follow-up: How do you invalidate that cache when an admin updates a policy?

**Answer:** When the admin API updates a policy in Postgres, it simultaneously publishes an invalidation event or directly deletes the cached key in Redis. The next request will experience a cache miss and fetch the fresh policy from Postgres.

---

### Bullet 3: "Scaled request handling via async task queues (RabbitMQ/Celery), isolating latency-sensitive API paths from persistence workloads with a common provider interface."

## Q. What exact tasks are you offloading to Celery/RabbitMQ?

**Answer:** Any work that does not need to block the immediate HTTP response to the client. Specifically: saving usage logs to PostgreSQL, executing the reconciliation of token budgets (refunds/deductions), triggering alerts for budget exhaustion, and updating the semantic cache in the background.

### Follow-up: Why use Celery and RabbitMQ? Why not just `asyncio.create_task()` or `BackgroundTasks` in FastAPI?

**Answer:** FastAPI's `BackgroundTasks` run in the same process memory. If the FastAPI pod crashes or restarts, all pending background tasks are instantly lost. Also, under high load, running heavy DB writes in the same event loop can starve the API workers, increasing latency. Celery + RabbitMQ provides durable queuing and allows us to scale API nodes and persistence workers independently.

### Follow-up: What is the message format you are sending to RabbitMQ?

**Answer:** It's a JSON-serialized payload containing the `request_id`, `org_id`, `provider`, `model`, `estimated_tokens`, `actual_tokens`, `latency_ms`, and `timestamp`. 

### Follow-up: What happens if the Celery worker is down?

**Answer:** RabbitMQ holds the messages in the queue. The API continues functioning and returning fast responses (isolating the critical path). Once the workers come back online, they drain the backlog. The only side effect is delayed usage analytics and delayed token reconciliation.

### Follow-up: How do you handle idempotency for usage logs if a task is retried?

**Answer:** Every request generates a unique UUID (`request_id`) at the API Gateway. The PostgreSQL `usage_logs` table has a unique constraint on `request_id`. If a Celery task retries and attempts to insert the same log, the DB throws a unique constraint violation, which the worker catches and ignores (or performs an UPSERT), ensuring we don't double-count usage.

---

## SECTION 2: ARCHITECTURE DEEP DIVE

## Q. Describe the full request lifecycle from the moment a user hits the gateway.

**Answer:** 
1. **Request arrives** at FastAPI endpoint.
2. **Auth dependency** extracts API key, validates it against the cache/DB, and identifies the Organization and User.
3. **Policy Engine** checks if this user is allowed to access the requested model/endpoint.
4. **Token Estimator** calculates expected input tokens using `tiktoken` (or a heuristic).
5. **Rate Limiter / Token Reserver** executes the Redis Lua script to atomically reserve budget. If rejected, returns 429.
6. **Semantic Cache** (optional) checks Redis (RediSearch) to see if this prompt was already answered. If hit, returns immediately.
7. **Router** selects the best provider/model and formats the prompt for the `LLMProvider` interface.
8. **Provider Call** makes the external HTTP call to OpenAI/Anthropic.
9. **Circuit Breaker** wraps the call. If the provider is failing, it trips and the router falls back to another provider.
10. **Response received**. Gateway normalizes the response format.
11. **Async Dispatch**: Gateway enqueues a Celery task with the actual token usage for reconciliation and DB logging.
12. **Response sent** to the user.

### Follow-up: What is strictly on the critical path for user latency?

**Answer:** Auth resolution (cached), Policy evaluation (cached), Token Estimation, Redis Lua script execution, and the actual external LLM HTTP call. 

### Follow-up: What happens if the Semantic Cache check takes too long?

**Answer:** The cache check should have a strict timeout (e.g., 50ms). If Redis is slow, we catch the `TimeoutError` and bypass the cache, proceeding to the real LLM call. The cache should be an optimization, not a point of failure.

---

## SECTION 3: TECHNOLOGY DEFENSE

### Redis (EXTRA DEEP)

## Q. You rely heavily on Redis. How does Redis execute Lua scripts?
**Answer:** Redis is primarily single-threaded for command execution. When you run `EVAL`, Redis blocks all other commands and executes the entire Lua script start to finish. This inherently guarantees atomicity without needing explicit locks.

### Follow-up: If it blocks all other commands, doesn't a slow Lua script kill your gateway throughput?
**Answer:** Yes. That's why the Lua script must be extremely small, doing only O(1) operations (like GET, SET, INCR, EXPIRE) and simple comparisons. No complex loops or heavy computation.

### Follow-up: Why not just use a distributed lock (Redlock) and regular GET/SET?
**Answer:** Because a lock requires multiple network round trips: acquire lock -> read budget -> calculate -> write budget -> release lock. That's 5 network hops. Under high concurrency, workers will spend all their time waiting for locks. Lua does it in 1 network hop.

### Follow-up: What is Redis persistence set to in your architecture?
**Answer:** By default, I use AOF (Append Only File) with `fsync=everysec`. This gives a good balance between performance (since disk writing is off the main thread) and durability (max 1 second of data loss on crash). 

### Follow-up: If your single Redis instance becomes a bottleneck, how do you scale it?
**Answer:** First, vertical scaling (bigger instance). Next, split concerns: one Redis for caching, one for rate limiting. If rate limiting still bottlenecks, we can use Redis Cluster.

### Follow-up: Wait, can you run your Lua script on a Redis Cluster?
**Answer:** Only if all keys accessed by the script map to the same hash slot. I enforce this by using curly braces `{}` in the key names, e.g., `budget:{org_123}:daily`. This ensures all budget keys for an organization hash to the same shard.

### PostgreSQL

## Q. Why use PostgreSQL for policies and usage logs instead of just keeping everything in Redis?
**Answer:** PostgreSQL provides relational integrity, complex querying, and durable storage. Usage logs grow rapidly and we need to query them by date, organization, user, and model for billing analytics. Redis is terrible for arbitrary multi-dimensional time-series queries over large datasets, whereas Postgres handles it well with proper indexing.

### Celery / RabbitMQ

## Q. What happens if RabbitMQ runs out of memory?
**Answer:** RabbitMQ will start blocking publishers (our FastAPI app) to relieve memory pressure, which would cause API requests to hang if not handled. We must set timeouts on our Celery `delay()` calls, and if the queue is unavailable, fail gracefully (perhaps falling back to writing a local emergency log file, or dropping the log if latency is paramount).

### Docker / Kubernetes / AWS

## Q. > **Needs candidate-specific confirmation** You mention Kubernetes and AWS. What exactly did you deploy?
**Answer:** I containerized the FastAPI app, Celery workers, and defined standard Kubernetes manifests (Deployments, Services, ConfigMaps). I deployed this on EKS (Elastic Kubernetes Service). *[Adjust based on actual truth: If just local minikube/docker-compose, state that frankly: "I used docker-compose for local orchestration and wrote K8s manifests targeting EKS, though production deployment was simulated."]*

---

## SECTION 4: CONCURRENCY TESTING DEEP DIVE

## Q. You mentioned 7,100+ concurrent tests. How exactly did you validate this?
**Answer:** I used Locust (`locustfile.py`). I created a scenario simulating thousands of users hitting the gateway simultaneously, all tied to the *same* organization API key, with a very tight budget (e.g., 10,000 tokens).

### Follow-up: What was the exact assertion that proved "no double charging"?
**Answer:** The test verified that exactly N requests returned a 200 OK (consuming the budget exactly to zero), and all subsequent concurrent requests returned a 429 Rate Limited. We then queried the Postgres usage logs and Redis final balance to assert that `sum(usage) == initial_budget` and `balance == 0`. Without the Lua script, the sum of usage far exceeded the initial budget.

### Follow-up: Did you find any failure modes during this load testing?
**Answer:** Yes. Initially, connection pooling to Postgres was exhausted because the Celery workers were opening too many connections to flush logs. I had to implement PgBouncer or tune SQLAlchemy's connection pool settings to handle the spike.

---

## SECTION 5: PROVIDER ABSTRACTION

## Q. How did you structure the `LLMProvider` interface?
**Answer:** It's an abstract base class (`abc.ABC`) with core asynchronous methods: `async def generate(...)` and `async def get_token_count(...)`. Every provider (OpenAI, Anthropic) implements this.

### Follow-up: How do you handle provider-specific quirks, like Claude not supporting certain OpenAI parameters?
**Answer:** The gateway defines a canonical, internal representation of a Request (e.g., standardizing system prompts, temperatures). The specific provider adapter is responsible for translating that canonical request into the provider's specific schema. If a feature isn't supported, the adapter either drops it (with a warning) or raises an exception.

---

## SECTION 6: DATABASE DEEP DIVE

## Q. What indexes do you have on the `usage_logs` table?
**Answer:** Since it's queried mostly for billing and analytics, there is an index on `org_id` and `timestamp`. The primary key is `request_id`.

### Follow-up: What happens when `usage_logs` hits 100 million rows? Querying `SUM(tokens)` will get very slow.
**Answer:** We would need to implement continuous aggregates or rollups. Instead of summing raw rows every time, a cron job (or Postgres trigger / materialized view) rolls up usage into daily/hourly buckets per organization.

---

## SECTION 7: FAILURE & DEBUGGING

## Q. The LLM provider (e.g., OpenAI) starts returning 500 Internal Server Errors. What exactly happens in your system?
**Answer:** 
1. The HTTP call fails.
2. The Circuit Breaker tracks the failure. After X consecutive failures, it trips to an "Open" state.
3. The Router detects the open circuit and immediately routes subsequent requests to the fallback provider (e.g., Anthropic).
4. The Catch block queues a Celery task to refund the tokens reserved for the failed OpenAI call back to the organization's budget.

### Follow-up: How does the Circuit Breaker ever close again?
**Answer:** It transitions to a "Half-Open" state after a timeout (e.g., 60 seconds). It allows a single test request through to OpenAI. If it succeeds, the circuit closes. If it fails, it trips open again.

---

## SECTION 8: SECURITY

## Q. How is multi-tenant isolation handled?
**Answer:** Every API key maps to an `org_id`. This `org_id` is injected into the request context (via standard FastAPI dependencies/middlewares). Every subsequent operation—Redis budget checks, Postgres queries, log writes—mandatorily uses this `org_id` as the primary partition key. 

### Follow-up: What if an API key is leaked?
**Answer:** The Gateway has a "Killswitch" module. Admins can invalidate an API key or an entire organization instantly in Postgres, which broadcasts a cache invalidation to Redis. Subsequent requests are blocked at the Auth dependency layer within milliseconds.

---

## SECTION 9: OWNERSHIP & ATTACK MODE

## Q. "You said you 'Scaled request handling via async task queues'. Did you actually face a scaling bottleneck, or did you just add Celery because it's a standard pattern?"
**Answer:** > **Needs candidate-specific confirmation** "Initially, I built it synchronously. During load testing with Locust, I observed that writing the usage logs to Postgres took 20-50ms. Under 1000 concurrent requests, this DB blocking caused the FastAPI event loop to stall, pushing P99 latency up by 400ms. By moving the log writing to Celery, I removed that DB wait time from the API critical path, bringing P99 latency back down to just the Redis + LLM network time."

---

## SECTION 10: WHAT WOULD YOU CHANGE?

## Q. Looking back at this architecture, what is the biggest flaw or bottleneck, and how would you fix it for V2?
**Answer:** The biggest flaw is that relying entirely on Redis for strict global token budgets becomes a bottleneck if we deploy across multiple geographic regions (e.g., US-East and EU-West). A strict atomic Lua script requires a single source of truth, meaning one region will suffer high cross-region latency. For V2, I would move to a relaxed coordination model (like a local budget token bucket that syncs with a global counter asynchronously), trading off strict atomicity for lower latency in multi-region deployments.
