## SECTION 1: RESUME BULLET INTERROGATION TREES

### Bullet 1: "Built a framework-agnostic experimentation and optimization platform for RAG, enabling automated evaluation and optimization of retrieval pipelines under production constraints."

## Q. Tell me about this RAG optimization platform. What problem does it solve?
**Answer:** It solves the "AutoML for RAG" problem. Instead of manually tweaking chunk sizes, k-values, or prompts, developers define latency and cost constraints, and RAGOS automatically explores configurations, evaluates them, and returns the optimal pipeline.

### Follow-up: How do you define a "framework-agnostic" platform here?
**Answer:** The core optimization engine does not depend on LlamaIndex or LangChain. It relies on internal Python interfaces (`interfaces.py`). Any retriever or embedder simply implements a common protocol, and the engine evaluates it identically.

### Follow-up: If it's framework-agnostic, how do you handle the wildly different parameters a LlamaIndex vector store needs vs a raw ChromaDB instance?
**Answer:** I use a plugin-based architecture and a configuration schema (`schema.py`). The optimizer generates a generic config dictionary, and the specific plugin adapter unpacks those parameters (like `top_k` or `similarity_threshold`) before passing them to the underlying implementation.

### Follow-up: How exactly does the system know which parameters are valid for a specific plugin?
**Answer:** > **Needs candidate-specific confirmation**. Each plugin registers its hyperparameter search space (e.g., bounds for chunk size, categorical choices for embedders) in `dimensions.py`. The optimizer only samples from these registered dimensions.

### Follow-up: What if a parameter combination is fundamentally invalid, like trying to use a sparse retriever with a dense embedding model?
**Answer:** The constraints engine (`constraints.py`) validates configurations before executing a trial. If a combo is invalid, it throws a `ConstraintViolationError` and the optimizer skips it or assigns it a zero score without running the expensive LLM calls.

### Follow-up: You mentioned "production constraints." What specific constraints are you modeling?
**Answer:** Primarily latency (milliseconds per query) and cost (dollars per 1000 tokens). If a pipeline configuration exceeds 2000ms p95 latency or $0.05 per query, it is penalized or discarded.

### Follow-up: How do you measure latency accurately when LLM API response times fluctuate wildly?
**Answer:** I calculate the mean and p90 latency over multiple query evaluations. For true production simulation, I strip out network jitter by mocking the LLM during purely retrieval-focused tests, but for end-to-end, I log actual time taken via `cost_tracker.py` and average it.

### Follow-up: Averages hide tail latencies. If an LLM hangs for 30 seconds once, does it ruin that config's score?
**Answer:** Yes, which is why I use p90 or p95 metrics rather than simple averages. If a config consistently hits tail latency issues, it fails the constraint check. 

---

### Bullet 2: "Implemented constraint-aware Random Search to optimize RAG pipelines (chunking, retrieval, generation) under latency and cost constraints, evaluating 100+ configurations."

## Q. Why Random Search instead of Grid Search or Bayesian Optimization?
**Answer:** Grid Search explodes combinatorially (e.g., 5 chunk sizes × 3 embedders × 4 top-k = 60 configs; adding one more dimension makes it 300). Random Search finds near-optimal solutions much faster in high-dimensional spaces because not all hyperparameters are equally important. Bayesian Optimization was too complex to implement initially given the categorical nature of many parameters (like choosing an LLM).

### Follow-up: You evaluated 100+ configurations. How long did that take end-to-end?
**Answer:** > **Needs candidate-specific confirmation**. Running 100 configurations over a dataset of 50 evaluation questions takes about [X hours]. The bottleneck is the LLM generation and evaluation steps.

### Follow-up: If the LLM is the bottleneck, how did you speed up the evaluation?
**Answer:** I utilized Python's `asyncio` to run independent trial queries concurrently. I also implemented semantic caching (`semantic_cache.py`) so identical LLM calls across different configs (e.g., same retrieved context and prompt) don't hit the API twice.

### Follow-up: Wait, if the retrieved context is identical, that implies the retrieval parameters were the same. When would that happen in random search?
**Answer:** Random search might pick the same chunk size and top-k, but change the generator LLM (e.g., Claude instead of GPT-4). The retrieval step is cached, saving embedder API calls and vector search time. Or if generation is the same but prompt changes, we still cache the retrieval output.

### Follow-up: How exactly does `semantic_cache.py` work? Is it an exact string match?
**Answer:** > **Needs candidate-specific confirmation**. It hashes the query and the exact retrieved context. If both match, it returns the cached generation. For purely semantic caching of queries, we'd use vector similarity, but for experiments, exact hashing of inputs ensures deterministic evaluation.

### Follow-up: What if the API rate limits you because of your async concurrency?
**Answer:** The `runner.py` uses `asyncio.Semaphore` to limit concurrent API calls. LiteLLM also handles basic retry logic with exponential backoff for HTTP 429 Too Many Requests errors.

---

### Bullet 3: "Implemented async experiment orchestration with automated evaluation, LiteLLM token cost profiling, and Pareto-frontier pipeline selection."

## Q. How does automated evaluation actually work in your system?
**Answer:** I use an LLM-as-a-judge approach (`evaluator.py`, `judge.py`). For each test query, the pipeline generates an answer. The judge LLM (usually GPT-4) is given the query, the generated answer, and a ground-truth reference, and outputs a score (1-5) for metrics like Contextual Precision, Answer Relevance, and Faithfulness.

### Follow-up: LLM-as-a-judge is notoriously fickle. How do you trust the scores?
**Answer:** I use strict prompting schemas requiring the judge to output step-by-step reasoning before the final integer score. I also use a highly capable model (GPT-4) for judging, and I validate the judge on a small hand-annotated dataset first to ensure alignment.

### Follow-up: How do you calculate the Pareto-frontier pipeline selection?
**Answer:** A configuration is on the Pareto frontier if no other configuration is strictly better across all target metrics (e.g., Accuracy and Latency). The algorithm (`pareto.py`) compares all evaluated configs O(N^2) and filters out any config that is "dominated" by another (lower accuracy AND higher latency).

### Follow-up: O(N^2) for Pareto is fine for 100 configs. What if you had 100,000?
**Answer:** I would sort the configurations first by one metric (e.g., Latency). Then iterate through, keeping a running maximum of the second metric (Accuracy). This reduces the Pareto extraction to O(N log N).

### Follow-up: How exactly does LiteLLM help with token cost profiling?
**Answer:** LiteLLM provides a unified interface for multiple LLM providers and automatically tracks token usage in its response objects. `cost_tracker.py` intercepts these responses, extracts `prompt_tokens` and `completion_tokens`, and multiplies them by the model's specific cost-per-token rates.

---

### Bullet 4: "Designed a plugin-based architecture enabling new retrievers, embedders, generators, and evaluators without modifying the core optimization engine."

## Q. Walk me through the code to add a new retriever to this plugin architecture.
**Answer:** You create a new Python class inheriting from the `BaseRetriever` interface in `interfaces.py`. You implement the `retrieve(query: str, top_k: int)` method. Finally, you register the class in the plugin registry (`plugins.py`) with a unique string identifier.

### Follow-up: How does the optimizer know what parameters this new retriever accepts?
**Answer:** The new retriever class must define a `get_hyperparameter_space()` class method (or static property) that returns a dictionary of its configurable dimensions. The optimizer dynamically reads this when constructing the search space.

### Follow-up: What if the new retriever requires a heavy initialization, like loading a local cross-encoder model into GPU memory?
**Answer:** The architecture uses lazy loading. Plugins are only instantiated when a trial requires them. However, if a trial iterates 10 times with the same model, I use a singleton pattern or connection pooling in the plugin registry so the model isn't reloaded every time.

### Follow-up: If it's a singleton, how do you handle concurrent async trials accessing the same cross-encoder?
**Answer:** Local GPU models don't naturally handle concurrent async requests well. I'd have to wrap the model inference in a thread pool using `asyncio.to_thread()`, or use a dedicated model serving endpoint (like vLLM) and have the plugin act as an HTTP client.

---

## SECTION 2: ARCHITECTURE DEEP DIVE

## Q. Describe the end-to-end architecture as if drawing on a whiteboard.
**Answer:** 
1. **Frontend/User:** A user submits an optimization job via the `/experiments` FastAPI endpoint, providing a dataset and constraints.
2. **Orchestrator:** FastAPI validates the request and passes it to the `Executor` (`executor.py`).
3. **Database:** The Executor creates an Experiment record in PostgreSQL.
4. **Optimizer:** The `Optimizer` reads the plugin registries and dimensions, generating N valid pipeline configurations.
5. **Runner:** The `Runner` executes these configurations asynchronously. For each config, it builds a `Pipeline` instance.
6. **Pipeline Execution:** 
   - Uses `Ingestion` to embed documents into ChromaDB.
   - Runs test queries through `Retriever`, `ContextBuilder`, and `Generator` (via LiteLLM).
7. **Evaluation:** `Evaluator` calls the judge LLM to score the outputs.
8. **Storage:** Results (scores, latency, cost) are saved back to PostgreSQL.
9. **Finalization:** The `Pareto` module calculates the best configs, updating the Experiment status to COMPLETED.

### Follow-up: Where is the state stored during the asynchronous trial runs?
**Answer:** In memory within the `Runner` loop, but intermediate results are periodically flushed to PostgreSQL. If the process crashes, we can resume because the completed trial IDs are persisted. ChromaDB holds the vector state temporarily per config (or persistently if namespaces are used).

### Follow-up: Wait, if 10 configurations have different chunk sizes, do you re-index the dataset into ChromaDB 10 times?
**Answer:** Yes. If the chunking strategy or embedding model changes, the vector store must be rebuilt. `vector_store.py` creates a unique isolated collection (or namespace) in ChromaDB for that specific config's indexing run, and tears it down after evaluation to save space.

### Follow-up: Re-indexing for every trial is extremely slow. How do you mitigate that?
**Answer:** I hash the ingestion parameters (chunk size, overlap, embedder). If a trial uses an ingestion hash that already exists in ChromaDB, the pipeline skips indexing and simply connects to that existing collection.

### Follow-up: What if the user submits a 10GB dataset? Your FastAPI endpoint will timeout while the optimizer runs.
**Answer:** The FastAPI endpoint doesn't wait. It's an async architecture. The POST `/experiments` creates the DB record, dispatches a background task (via FastAPI `BackgroundTasks` or Celery/Redis), and returns a `202 Accepted` with a Job ID. The user polls GET `/experiments/{id}` for status.

---

## SECTION 3: TECHNOLOGY DEFENSE

### 1. FastAPI
- **L1 Basic:** What is it? A modern, fast web framework for building APIs with Python based on standard Python type hints.
- **L2 Project:** Why here? Needed a lightweight backend to expose the optimization engine, handle async requests natively, and generate OpenAPI docs automatically.
- **L3 Mechanism:** Uses Starlette for the web parts and Pydantic for the data parts. It handles concurrency using Python's `async def`.
- **L4 Tradeoffs:** Why not Flask/Django? Flask is synchronous by default and lacks built-in Pydantic validation. Django is too heavy for a microservice focused on running RAG pipelines.
- **L5 Failure:** What if FastAPI crashes? The web server (Uvicorn) can use process managers (like Supervisor or Docker restart policies) to spin back up, but any in-memory background tasks will be lost.
- **L6 Debugging:** If an endpoint hangs, I'd check if a synchronous blocking call (like a slow DB query or synchronous LLM call) was made inside an `async def` function, which blocks the event loop.
- **L7 Scaling:** At 10x load, I'd scale Uvicorn workers and deploy behind a load balancer (Nginx/ALB) across multiple Docker containers. Background tasks would need to move to Celery/RabbitMQ.

### 2. PostgreSQL
- **L1 Basic:** An open-source relational database.
- **L2 Project:** Stores experiment metadata, configurations, constraints, and evaluation scores.
- **L3 Mechanism:** Uses structured tables with strict typing and ACID compliance. Accessed via an ORM (like SQLAlchemy) or async driver (asyncpg).
- **L4 Tradeoffs:** Why not MongoDB? Experiment metadata and results are highly structured and relational (Experiment -> Trials -> Scores). SQL allows complex JOINs for analytics and Pareto calculations.
- **L5 Failure:** If Postgres goes down, new experiments fail to save. The application throws a 500.
- **L6 Debugging:** If queries are slow, I'd use `EXPLAIN ANALYZE` to check if the `trials` table lacks an index on `experiment_id` or `score`.
- **L7 Scaling:** Add read replicas for querying analytics. Partition the `trials` table by date or experiment if it grows to millions of rows.

### 3. ChromaDB
- **L1 Basic:** An open-source vector database.
- **L2 Project:** Stores chunked document embeddings and performs nearest-neighbor search during the retrieval phase of the RAG pipeline.
- **L3 Mechanism:** Uses HNSW (Hierarchical Navigable Small World) graphs under the hood to perform fast approximate nearest neighbor (ANN) search.
- **L4 Tradeoffs:** Why not Pinecone/Milvus? ChromaDB can run entirely locally in-memory or on disk (via SQLite/Parquet), which is perfect for isolating temporary experiment vector stores without incurring network latency or cloud costs.
- **L5 Failure:** If ChromaDB corrupts its local SQLite file, the vector space is lost. Trials would crash.
- **L6 Debugging:** If retrieval returns garbage, I'd manually query the collection to ensure embeddings were actually generated and not just empty arrays, and check the distance metric (cosine vs L2).
- **L7 Scaling:** Chroma local doesn't scale horizontally. I'd migrate to a distributed vector DB like Milvus, Qdrant, or Pinecone.

### 4. LiteLLM
- **L1 Basic:** A library that standardizes API calls to 100+ LLMs using the OpenAI format.
- **L2 Project:** Allows the optimizer to swap out OpenAI, Gemini, and Ollama dynamically without writing custom API client code for each trial.
- **L3 Mechanism:** It intercepts the standardized request, translates the prompt/params to the provider's specific format, makes the HTTP call, and normalizes the response and token usage.
- **L4 Tradeoffs:** Why not LangChain's LLM wrappers? LangChain is too heavy, abstract, and slow. LiteLLM is a thin, fast proxy focused solely on standardizing the I/O.
- **L5 Failure:** If LiteLLM's internal mappings are outdated due to a provider API change, the call fails.
- **L6 Debugging:** Enable LiteLLM's verbose logging to see the exact raw HTTP payload being sent to the provider.
- **L7 Scaling:** LiteLLM is stateless. Scaling just means running more instances. We can also use LiteLLM Proxy for centralized rate limiting and key management.

### 5. Docker
- **L1 Basic:** A platform for developing, shipping, and running applications in isolated containers.
- **L2 Project:** Packages the FastAPI app, dependencies, and environment variables into a reproducible image.
- **L3 Mechanism:** Uses Linux cgroups and namespaces to isolate processes. A Dockerfile defines the build steps.
- **L4 Tradeoffs:** Why not just bare metal? Dependency hell. Python environments, ChromaDB C++ dependencies, and specific library versions are bundled safely.
- **L5 Failure:** Container OOM (Out of Memory) crash.
- **L6 Debugging:** Run `docker logs` to see if FastAPI threw an exception. Run `docker stats` to check if ChromaDB is eating all RAM.
- **L7 Scaling:** Deploy the containers via Kubernetes or AWS ECS.

### 6. AsyncIO (Python)
- **L1 Basic:** A library to write concurrent code using the async/await syntax.
- **L2 Project:** Used to execute multiple LLM API calls and trials concurrently, overcoming the network I/O bottleneck.
- **L3 Mechanism:** Uses an event loop. When an I/O operation (like `httpx.get`) is awaited, the loop pauses that coroutine and switches to another one that is ready.
- **L4 Tradeoffs:** Why not Threading/Multiprocessing? Threads have high memory overhead and GIL contention. Multiprocessing is heavy. AsyncIO is lightweight and perfect for heavily I/O-bound tasks like API calls.
- **L5 Failure:** If a developer puts a synchronous block (like `time.sleep` or a massive CPU calculation) in a coroutine, the entire event loop freezes.
- **L6 Debugging:** Enable asyncio debug mode (`PYTHONASYNCIODEBUG=1`) to log coroutines that take too long to execute.
- **L7 Scaling:** Asyncio handles thousands of concurrent connections on a single core. To scale further, run multiple event loops across multiple cores using `uvicorn --workers N`.

---

## SECTION 4: CONCEPT DEFENSE

### Concept: Pareto Frontier

## Q. You mentioned finding the Pareto-frontier. Explain what that means in the context of RAG.
**Answer:** In RAG optimization, you have competing objectives, typically Accuracy (maximize) and Latency/Cost (minimize). A configuration is on the Pareto frontier if it is impossible to improve one metric without worsening another. The frontier is the set of all such optimal trade-off configurations.

### Follow-up: How do you mathematically determine if Config A dominates Config B?
**Answer:** Config A dominates Config B if A is strictly better than B in at least one metric, and greater than or equal to B in all other metrics. (e.g., A has higher accuracy and lower latency).

### Follow-up: What if Config A has 95% accuracy and $0.10 cost, and Config B has 94% accuracy and $0.01 cost. Who dominates?
**Answer:** Neither. A is better in accuracy; B is better in cost. Both belong on the Pareto frontier. The user must choose based on their business preference.

### Follow-up: How do you handle noise? If Accuracy varies by +/- 2% between runs, your frontier might be calculating based on statistical noise.
**Answer:** > **Needs candidate-specific confirmation**. I would compute confidence intervals for the metrics across multiple query runs. If the difference between A and B is not statistically significant (p > 0.05 via a t-test), I treat them as tied for that metric, or I run more evaluations to reduce the variance.

---

### Concept: Random Search vs Bayesian

## Q. Why did you build Random Search instead of Bayesian Optimization?
**Answer:** Bayesian Optimization assumes a continuous, smooth objective function. RAG hyperparameters are highly categorical and discontinuous (e.g., swapping `BM25` for `VectorSearch`, or `GPT-3.5` for `Claude`). Random search handles categorical variables natively, is embarrassingly parallelizable, and is a strong baseline.

### Follow-up: But random search explores useless space. How do you prevent it from trying terrible configs?
**Answer:** By using the Constraints Engine. The search space is bounded by the plugin dimensions, and any randomly generated config must pass validation rules (e.g., `chunk_overlap` must be `< chunk_size`) before execution.

---

## SECTION 5: DATABASE DEEP DIVE

## Q. Why use both PostgreSQL and ChromaDB?
**Answer:** They serve entirely different purposes. PostgreSQL is an ACID-compliant relational database used to store the application state: Experiment IDs, user configurations, constraints, and the final evaluation scores (structured data). ChromaDB is a specialized vector database used during the trial execution to store text chunks and high-dimensional embeddings for nearest-neighbor retrieval (unstructured data).

### Follow-up: Detail the schema for PostgreSQL. What tables do you have?
**Answer:** 
- `experiments`: id, status, dataset_id, created_at, constraints_json
- `trials`: id, experiment_id, config_json, metrics_json, status, error_logs
- `datasets`: id, name, raw_text_path

### Follow-up: Which queries are common, and which could be slow?
**Answer:** Common: `INSERT INTO trials`, `UPDATE trials SET status`.
Slow: Calculating the Pareto frontier if done in SQL, requiring self-joins: `SELECT * FROM trials t1 WHERE NOT EXISTS (SELECT 1 FROM trials t2 WHERE t2.accuracy > t1.accuracy AND t2.latency < t1.latency)`.

### Follow-up: If you had millions of trial records, how would you optimize that Pareto query?
**Answer:** I wouldn't do it in SQL. I would query `SELECT config, accuracy, latency FROM trials WHERE experiment_id = X`, load it into Python (pandas/numpy), and calculate the frontier in memory using an O(N log N) sweep-line algorithm.

---

## SECTION 6: API DEEP DIVE

## Q. Walk me through the POST `/experiments` endpoint.
**Answer:** 
Request Payload: `dataset_id`, `search_space` (ranges for chunk size, list of models), `constraints` (max latency, min accuracy), `num_trials` (e.g., 50).
Action: Validates payload with Pydantic. Inserts a new row in `experiments` table with status PENDING. Triggers `runner.execute_experiment(id)` as a FastAPI background task.
Response: Returns HTTP 202 Accepted with `{"experiment_id": 123, "status": "running"}`.

### Follow-up: What happens if the user passes `min_accuracy: 1.5` when accuracy is bounded 0.0 to 1.0?
**Answer:** Pydantic catches this during request parsing using a `Field(ge=0.0, le=1.0)` validator. It immediately returns a HTTP 422 Unprocessable Entity with a clear error message, before touching the database.

### Follow-up: How do you stop a runaway experiment?
**Answer:** I would need a POST `/experiments/{id}/cancel` endpoint. It sets the DB status to CANCELLED. The async runner loop checks the DB status before starting the next trial; if cancelled, it breaks the loop and cleans up ChromaDB.

---

## SECTION 7: PERFORMANCE DEFENSE

## Q. You evaluated 100+ configurations. Where was the primary bottleneck?
**Answer:** The LLM generation and evaluation steps. If I test 100 configs against 50 queries, that's 5,000 generation calls and 5,000 evaluation calls. Even at 1 second per call, that's almost 3 hours sequentially.

### Follow-up: How did you optimize this?
**Answer:** 
1. **Async Concurrency:** Ran queries concurrently using `asyncio.gather` with a semaphore of 20 to respect API rate limits.
2. **Semantic Caching:** If Config A and Config B use the same retrieval settings but different generators, the retrieval step isn't repeated.
3. **Early Stopping:** If a config's average latency crosses the constraint threshold after 5 queries, I abort the remaining 45 queries and mark the trial as FAILED_CONSTRAINT.

### Follow-up: Early stopping based on 5 queries? What if the first 5 were just unusually slow network spikes?
**Answer:** I check if the moving average of latency plus the standard deviation safely exceeds the constraint. If it's borderline, I continue. I only aggressively stop if the p90 latency is an order of magnitude higher than allowed (e.g., 10 seconds vs 1 second constraint).

---

## SECTION 8: FAILURE & DEBUGGING SCENARIOS

## Q. What happens if ChromaDB runs out of memory during a massive document ingestion step?
**Answer:** The Chroma client throws an exception. The `ingest.py` module catches it, logs the error, marks that specific Trial as FAILED in PostgreSQL with the error stack trace, and the optimizer seamlessly moves on to the next configuration.

## Q. LLM API returns HTTP 500 errors mid-experiment. How do you handle it?
**Answer:** `LiteLLM` has built-in retry logic (Tenacity). It will retry with exponential backoff. If it still fails after 3 retries, the specific evaluation query fails. I can either fail the whole trial, or record a null score for that query and compute the average over the successful ones, depending on strictness.

## Q. Evaluation scores are wildly inconsistent across runs for the exact same configuration.
**Answer:** This means the LLM Judge is non-deterministic. Debugging steps:
1. Ensure temperature is set to 0.0 for the judge LLM.
2. Check the prompts; they might be too ambiguous. I'd switch to a stricter CoT (Chain of Thought) prompt.
3. Upgrade the judge model (e.g., from GPT-3.5 to GPT-4o).

---

## SECTION 9: OWNERSHIP QUESTIONS

## Q. What exactly did YOU build vs what did libraries handle?
**Answer:** Libraries (LiteLLM, Chroma) handled API parsing and vector math. I built the Orchestration engine (`runner.py`), the Optimizer logic (Random Search with Constraints), the Plugin architecture, the Evaluation pipelines, and the Pareto-frontier calculation. I glued the frameworks together into a cohesive AutoML product.

## Q. What was the hardest technical challenge?
**Answer:** Managing async state and isolation. Ensuring that Trial 1's ChromaDB vectors didn't bleed into Trial 2's retrieval step, while running them concurrently, required dynamic collection namespacing and careful dependency injection in the `Pipeline` class.

---

## SECTION 10: INTERVIEWER ATTACK MODE

## Q. "You said framework-agnostic, but your code imports `chromadb`. That's not agnostic."
**Answer:** The core engine doesn't import ChromaDB. `vector_store.py` in the core defines an abstract base class. The `plugins/chroma_adapter.py` imports ChromaDB. If I want to use Pinecone, I write a new adapter. The engine only knows about `vector_store.search()`.

## Q. "You said 100+ configurations. Is that statistically significant to find a global optimum?"
**Answer:** Not for a global optimum. Random Search does not guarantee a global optimum. However, empirical studies in hyperparameter tuning (like Bergstra and Bengio, 2012) show that Random Search can find configurations within 5% of the optimum using just 60 trials, because usually only a few dimensions truly matter.

## Q. "How reliable is your LLM-as-a-judge? Did you measure correlation with human scores?"
**Answer:** > **Needs candidate-specific confirmation**. In a production setup, I would run a baseline set of 100 queries evaluated by humans, run the LLM judge on them, and calculate the Pearson or Spearman correlation coefficient. If correlation > 0.8, the judge is trusted.

---

## SECTION 11: WHAT WOULD YOU CHANGE?

## Q. If you had 3 more months to work on this, what would you rebuild?
**Answer:** 
1. **Bayesian Optimization / Optuna:** I would integrate Optuna to use Tree-structured Parzen Estimator (TPE) instead of naive random search to converge faster.
2. **Distributed Execution:** I would move from `asyncio` on a single machine to a distributed task queue (Celery/Redis or Ray) so I can scale workers across multiple EC2 instances.
3. **Advanced RAG Patterns:** I would add support for complex pipelines like GraphRAG, Multi-Agent routing, and query decomposition, which require optimizing a DAG of tasks rather than a linear pipeline.
4. **Better Evaluation:** I'd integrate RAGAS (Retrieval Augmented Generation Assessment) directly, as they have mathematically rigorous definitions for faithfulness and answer relevance.
