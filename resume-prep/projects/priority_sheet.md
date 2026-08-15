# AI/Backend Interview Priority Sheet

> Prepare only claims you can defend from your own implementation or evidence. Where this sheet names a tool, behavior, or metric, treat it as a verification prompt—not a line to memorize if it was not actually part of your project.

## Scoring Formula
Score = P(asked) × Depth(1-3) × Risk(1-3)
P(asked): 0.3=unlikely, 0.6=possible, 0.9=very likely
Depth: 1=surface, 2=moderate, 3=deep dive expected
Risk: 1=recoverable, 2=awkward, 3=immediate credibility damage

## 🔴 Must Know Perfectly (Top 10)

### Q1. [Score: 8.1] Concurrent Budget Enforcement (TOCTOU)
**Question:** Explain the exact race condition you solved in LLM Cost Guard.
**Why ranked here:** It's the core technical meat of your strongest project; if you can't explain the bug, the whole project sounds fake.
**Perfect answer in 3 bullets:**
- Two requests check the budget simultaneously; both see sufficient funds.
- Both pass the check and decrement, resulting in a negative balance.
- This is a Time-Of-Check to Time-Of-Use (TOCTOU) race condition.
**If you blank:** "Imagine two people with the same debit card withdrawing the last $10 at two different ATMs at the exact same millisecond."

### Q2. [Score: 8.1] Redis Lua Atomicity
**Question:** How exactly does a Redis Lua script fix the race condition?
**Why ranked here:** Direct follow-up to Q1. You must know why your solution works.
**Perfect answer in 3 bullets:**
- Redis is single-threaded for command execution.
- A Lua script blocks all other commands while it runs.
- By putting the read (check) and write (decrement) in one script, it becomes a single atomic operation.
**If you blank:** "Redis executes commands one by one. The Lua script wraps the check and math into one unbreakable step."

### Q3. [Score: 8.1] Shannon Entropy on Frame Differences
**Question:** How did you use Shannon Entropy to reduce YOLO compute by 93%?
**Why ranked here:** It's the most mathematically interesting part of Focus Lock and highlights your optimization skills.
**Perfect answer in 3 bullets:**
- I took the absolute difference between two consecutive frames to isolate motion.
- I calculated the Shannon Entropy of that difference image; no motion = low entropy.
- If entropy was below a threshold, I skipped the heavy YOLO inference entirely.
**If you blank:** "I looked at the difference between frames. If the math showed it was mostly uniform black pixels, I knew nothing moved, so I skipped YOLO."

### Q4. [Score: 5.4] RAGOS Constraint Filtering
**Question:** How did you filter configurations before the trial in RAGOS?
**Why ranked here:** It's the stated "interesting part" of the project in your intro.
**Perfect answer in 3 bullets:**
- Evaluated theoretical max cost (tokens * rate) and estimated latency.
- Discarded configurations that mathematically couldn't meet user constraints.
- Prevented wasting expensive API calls on doomed configurations.
**If you blank:** "I used a simple math formula to estimate the cost based on chunk size and model price before hitting the API."

### Q5. [Score: 5.4] System Architecture: Proxy / Gateway
**Question:** Why build a proxy instead of just putting the budget logic in the client app?
**Why ranked here:** Tests foundational system design understanding.
**Perfect answer in 3 bullets:**
- Centralized enforcement across multiple different client applications.
- Prevents malicious or buggy clients from bypassing limits.
- Allows unified tracking and API key management in one secure place.
**If you blank:** "A proxy ensures all traffic goes through one checkpoint, making it impossible for a rogue app to overspend."

### Q6. [Score: 5.4] Python GIL & Threading
**Question:** How did your 4-thread model in Python actually run concurrently given the GIL?
**Why ranked here:** You explicitly mentioned GIL interaction in Focus Lock; interviewers love GIL questions.
**Perfect answer in 3 bullets:**
- The GIL prevents pure Python bytecode from running in parallel on multiple cores.
- However, heavy libraries like OpenCV and YOLO run in C extensions.
- These extensions release the GIL, allowing my Python I/O threads (like WebSocket) to run concurrently.
**If you blank:** "The heavy lifting was in C++, which drops the GIL, so my Python UI thread wasn't blocked."

### Q7. [Score: 5.4] Locust Load Testing Validation
**Question:** How did you prove the TOCTOU bug was fixed?
**Why ranked here:** Proves you know how to validate backend reliability.
**Perfect answer in 3 bullets:**
- Simulated high concurrency using Locust with mock API endpoints.
- Fired requests totaling more than the allotted budget simultaneously.
- Asserted that exactly the budget amount succeeded, the rest 429'd, and the final balance was zero.
**If you blank:** "I blasted the local server with concurrent requests and checked that the database never went below zero."

### Q8. [Score: 5.4] Redis vs. PostgreSQL
**Question:** Why use Redis for the budget checks when you also have PostgreSQL?
**Why ranked here:** Standard database trade-off question tied to your stack.
**Perfect answer in 3 bullets:**
- PostgreSQL is great for durable, relational storage (users, policies).
- Budget checks happen on every single request and require microsecond latency.
- Redis's in-memory speed and atomic scripts are better suited for high-frequency rate limiting.
**If you blank:** "Postgres is for long-term storage; Redis is for fast, volatile counting."

### Q9. [Score: 3.6] Random Search vs. Grid Search
**Question:** Why Random Search instead of Grid Search in RAGOS?
**Why ranked here:** Basic optimization knowledge required for the RAGOS project.
**Perfect answer in 3 bullets:**
- Grid search suffers from the curse of dimensionality.
- Random search explores the space more effectively because not all parameters are equally important.
- It tests unique values on important dimensions rather than repeating them.
**If you blank:** "Grid search tests too many useless combinations. Random search finds good enough answers much faster."

### Q10. [Score: 3.6] Pareto Frontier
**Question:** What does returning a Pareto frontier mean?
**Why ranked here:** You namedrop it; you must know the definition.
**Perfect answer in 3 bullets:**
- It's the set of all optimal configurations.
- A point is on the frontier if you cannot improve cost without degrading latency/accuracy (non-dominated).
- It gives the user the best trade-offs rather than forcing a single arbitrary weight.
**If you blank:** "It's a graph showing the best possible trade-offs, where no option is strictly worse than another in every way."

## 🟡 Must Understand Conceptually (Q11–Q20)

### Q11. [Score: 3.6] Semantic Caching
**Question:** How does your semantic cache work?
**Why ranked here:** High buzzword value on resume.
**Perfect answer in 3 bullets:**
- Embeds incoming queries.
- Compares against stored embeddings using cosine similarity.
- Returns cached response if similarity > threshold.

### Q12. [Score: 3.6] Circuit Breaker Pattern
**Question:** How did you implement a circuit breaker?
**Why ranked here:** Key reliability pattern.
**Perfect answer in 3 bullets:**
- Monitors failure rates of upstream providers (e.g. OpenAI goes down).
- Trips 'open' to fail fast and stop overwhelming the provider.
- 'Half-open' allows a few test requests to see if it recovered.

### Q13. [Score: 2.4] Distributed Message Queues (RabbitMQ/Celery)
**Question:** Why use RabbitMQ/Celery for async reconciliation?
**Why ranked here:** Standard backend pattern.
**Perfect answer in 3 bullets:**
- Decouples the fast Redis check from the slow Postgres update.
- Provides retries and guarantees delivery if Postgres is temporarily down.
- Smooths out write spikes.

### Q14. [Score: 2.4] MediaPipe vs. YOLO
**Question:** Why use both YOLO and MediaPipe?
**Why ranked here:** Architecture question for Focus Lock.
**Perfect answer in 3 bullets:**
- YOLO is great for general object detection (phones).
- MediaPipe is highly optimized specifically for facial landmarks and head pose.
- Combining them gave the best of both worlds.

### Q15. [Score: 2.4] YOLOv8 Anchor-Free Architecture
**Question:** What makes YOLOv8 "anchor-free"?
**Why ranked here:** You wrote it on your resume.
**Perfect answer in 3 bullets:**
- Predicts bounding box centers directly.
- Eliminates the need to hand-tune anchor box sizes before training.
- Makes the model simpler and more flexible.

### Q16. [Score: 2.4] RAG Architecture
**Question:** What are the components of a standard RAG pipeline?
**Why ranked here:** Context for RAGOS.
**Perfect answer in 3 bullets:**
- Document chunking/embedding.
- Vector database retrieval (ChromaDB).
- LLM generation using the retrieved context.

### Q17. [Score: 2.4] Asynchronous Python (FastAPI)
**Question:** Why use FastAPI over Flask for LLM Cost Guard?
**Why ranked here:** Framework choice justification.
**Perfect answer in 3 bullets:**
- Native ASGI support for `async`/`await`.
- Ideal for I/O bound proxy workloads waiting on external APIs.
- Automatic OpenAPI docs.

### Q18. [Score: 1.8] LLM-as-a-Judge Evaluation
**Question:** How did you evaluate RAG configurations automatically?
**Why ranked here:** Crucial for AutoML in RAG.
**Perfect answer in 3 bullets:**
- Used a stronger model (like GPT-4) to evaluate outputs.
- Measured context relevance and faithfulness.
- Standardized prompts for consistent scoring.

### Q19. [Score: 1.8] Finite State Machines (FocusFSM)
**Question:** How did you prevent state flickering in Focus Lock?
**Why ranked here:** Practical software engineering challenge.
**Perfect answer in 3 bullets:**
- Modeled states explicitly (IDLE, FOCUSED, DISTRACTED).
- Used debouncing (must hold state for N frames to transition).
- Handled transitions strictly based on defined rules.

### Q20. [Score: 1.8] Adaptive Sampling / Edge Cases
**Question:** What is the "frozen-phone" problem?
**Why ranked here:** Shows deep thought about edge cases.
**Perfect answer in 3 bullets:**
- If someone holds a phone perfectly still, entropy is zero.
- The system might skip YOLO and assume they are focused.
- Solved by forcing a periodic inference regardless of entropy.

## 🟢 Nice to Know (Q21–Q30)

### Q21. [Score: 1.2] WebSockets vs HTTP
**Question:** Why use WebSockets in Focus Lock?
**Why ranked here:** Networking basics.
**Perfect answer in 3 bullets:**
- Bidirectional, persistent connection.
- Lower latency for real-time state updates to the UI.
- Avoids HTTP polling overhead.

### Q22. [Score: 1.2] SITL / ArduPilot
**Question:** What is SITL in your research?
**Why ranked here:** Drone research context.
**Perfect answer in 3 bullets:**
- Software In The Loop.
- Runs the exact flight controller code on a PC.
- Simulates physics to test without crashing real hardware.

### Q23. [Score: 0.9] ChromaDB / Vector Search
**Question:** How does vector search work fundamentally?
**Why ranked here:** Good database knowledge.
**Perfect answer in 3 bullets:**
- Converts text to high-dimensional embeddings.
- Uses Approximate Nearest Neighbor (ANN) like HNSW.
- Finds vectors close in semantic space.

### Q24. [Score: 0.9] OpenCV Feature Tracking
**Question:** How did you track underwater features?
**Why ranked here:** Computer vision basics.
**Perfect answer in 3 bullets:**
- Found keypoints (corners/edges).
- Computed descriptors invariant to scale/rotation.
- Matched features between frames.

### Q25. [Score: 0.9] SQLite
**Question:** Why SQLite for Focus Lock?
**Why ranked here:** Database selection.
**Perfect answer in 3 bullets:**
- Serverless, local file-based DB.
- Perfect for a local desktop app.
- Zero setup required for the user.

### Q26. [Score: 0.9] Cordum Policy Callback (Accurate Contribution)
**Question:** What did your Cordum PR actually implement?
**Why ranked here:** The public PR is inspectable, so accuracy matters more than an impressive story.
**Perfect answer in 3 bullets:**
- Added a LangChain `govern()` helper and callback-based policy check.
- Called `/api/v1/policy/evaluate` through `httpx` before tool execution.
- Supported sync and async callback paths; deny/approval decisions raise `PermissionError`.

### Q27. [Score: 0.9] Open-Source Honesty
**Question:** Was the Cordum PR merged, and what did you learn?
**Why ranked here:** A direct public-source check can expose an inaccurate answer.
**Perfect answer in 3 bullets:**
- PR #263 was closed, not merged; do not claim the later merged #264 as yours.
- Explain the two files and 142 lines you added, plus LCEL/legacy callback compatibility.
- Say the later design credited the PR as inspiration, then move to what you learned from the integration.

### Q28. [Score: 0.9] Hardware / C++ Integration
**Question:** How do Python and C++ interact in OpenCV?
**Why ranked here:** Systems knowledge.
**Perfect answer in 3 bullets:**
- Python uses C API wrappers (like pybind11).
- Python passes pointers/buffers to C++.
- C++ does the math fast and returns results.

### Q29. [Score: 0.9] LiteLLM
**Question:** What is LiteLLM doing under the hood?
**Why ranked here:** Tooling knowledge.
**Perfect answer in 3 bullets:**
- Translates standard OpenAI format to other provider formats.
- Handles retries and fallbacks.
- Simplifies the codebase significantly.

### Q30. [Score: 0.9] Multi-threading vs Multi-processing
**Question:** When would you use multiprocessing instead of threading?
**Why ranked here:** Python specific concurrency.
**Perfect answer in 3 bullets:**
- Use multiprocessing for CPU-bound pure Python code.
- It bypasses the GIL by creating entirely new processes.
- High memory overhead compared to threads.

## Study Schedule Recommendation
1. **Days 1-2 (The Core - Q1-Q4):** Spend 80% of your time mastering the TOCTOU race condition, Redis Lua, and Shannon Entropy. Practice drawing the race condition on a whiteboard.
2. **Day 3 (System Design - Q5-Q8):** Review the proxy architecture, database choices, and GIL threading mechanics.
3. **Day 4 (Machine Learning / RAG - Q9-Q12):** Ensure you can explain Random Search, Pareto, and semantic caching clearly.
4. **Day 5 (Mock Interviews):** Have a friend specifically grill you on the Top 10 using the L4/L5 questions from the `intro_interview_tree.md` document.
5. **Day 6 (Review):** Skim the "Nice to Know" section just so you don't blank if they ask a random question about SQLite or WebSockets.
