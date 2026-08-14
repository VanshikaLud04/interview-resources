# PART 1: Cordum Open Source Contribution

## Q. Tell me about your open source contribution to Cordum.

**Answer:** I contributed to Cordum by redesigning their policy enforcement mechanism. I moved it from a synchronous, blocking HTTP architecture to an asynchronous job pipeline, which eliminated blocking evaluations from the core execution path.

### Follow-up: What was the problem with the existing architecture?

**Answer:** Policy enforcement was completely synchronous. Every time an LLM execution was triggered, the thread had to wait for an HTTP round-trip to the policy service to return an approval before proceeding, adding blocking latency to every single request.

### Follow-up: Why is blocking bad here specifically?

**Answer:** LLM calls are already extremely high-latency, often taking seconds. Adding a synchronous HTTP policy check before each call compounds this delay and introduces a tight coupling point where the policy service becomes a bottleneck and a single point of failure for the entire application.

### Follow-up: What exactly is an async job pipeline in this context?

**Answer:** Instead of executing the policy check in the main request thread, the task is offloaded to a background queue. The policy evaluation is enqueued as a discrete job, allowing the main application to continue its work, accept other requests, or immediately return a pending state while a background worker processes the policy check independently.

### Follow-up: If the pipeline is async, how does the system know when to block the LLM if the policy is violated?

**Answer:** The execution state is decoupled. The main service receives an acknowledgment that the job is queued and tracks a state object. The LLM execution router checks this state asynchronously. If the sidecar processing the job determines a denial, it updates the state. When the router yields back to check the state, it routes to a 403 handler if denied.

### Follow-up: What happens if the sidecar crashes while a job is in the pipeline?

**Answer:** The system relies on message acknowledgment. If the sidecar crashes before completing the evaluation, the job remains unacknowledged in the queue. A retry mechanism or a dead-letter queue handles it, while the main service relies on a timeout to eventually fail the request gracefully rather than hanging forever.

### Follow-up: How do you handle the tradeoff of eventual consistency here?

**Answer:** The tradeoff is that a request might sit in a "pending" state longer if the queue is backed up, rather than failing immediately. We accept this because overall system throughput increases. The system handles this by ensuring the LLM is never invoked until a positive terminal state is reached.

## Q. What is a sidecar pattern?

**Answer:** A sidecar is an architectural deployment pattern where a helper service runs alongside the primary application, typically in the same pod or on the same host machine. It handles ancillary, isolated tasks—in this case, policy evaluation—without modifying the main application's codebase.

### Follow-up: Why use a sidecar instead of just embedding the logic in the main service?

**Answer:** Separation of concerns, fault isolation, and independent scalability. Embedding it tightly couples the policy logic and dependencies with the core app. A sidecar ensures that a memory leak or crash in the heavy ML-based policy evaluator doesn't take down the core API serving traffic.

### Follow-up: Why a sidecar instead of middleware?

**Answer:** Middleware executes within the same process and runtime as the host application. A sidecar is an out-of-process architecture. We needed a sidecar to isolate the compute-heavy python evaluation logic from the main application's event loop, preventing CPU-bound tasks from blocking async I/O.

### Follow-up: How exactly does the sidecar communicate with the main service?

**Answer:** > **Needs candidate-specific confirmation** (e.g., HTTP over localhost, gRPC, or communicating indirectly via a shared Redis queue).

### Follow-up: Why use Python for the sidecar?

**Answer:** Python is the undisputed standard for the AI/ML ecosystem. Writing the `cordum-langchain-guard` in Python allowed native, frictionless integration with LangChain's evaluation libraries and ML tooling, avoiding complex foreign function interfaces or sub-par ports in other languages.

### Follow-up: Could this sidecar be written in Go? What would be the tradeoff?

**Answer:** Yes, rewriting it in Go would vastly improve performance, memory footprint, and concurrency handling. However, the critical tradeoff is ecosystem maturity. We would lose native access to Python-based LLM orchestration libraries, forcing us to write custom wrappers or use less mature Go equivalents.

## Q. What is a "deterministic 403"?

**Answer:** It means the system guarantees that any policy violation strictly and predictably results in an HTTP 403 Forbidden status, without side effects, race conditions, or leaking sensitive backend information. The failure mode is highly controlled and testable.

### Follow-up: What exactly were the 26 integration tests doing?

**Answer:** They were testing the entire lifecycle boundary: sending API requests, verifying the async job was accurately created in the queue, simulating both approvals and denials from the sidecar, and asserting that the final system state accurately routed to a 403 or execution success.

### Follow-up: How do you write integration tests for an asynchronous workflow without flaky tests?

**Answer:** You avoid arbitrary `sleep()` calls. Instead, you use polling with timeouts or event synchronization primitives. The test triggers the workflow and polls the status endpoint or database until a terminal state (approval/denial) is reached, failing if it exceeds a maximum duration.

### Follow-up: What feedback did the maintainers give during the PR review?

**Answer:** > **Needs candidate-specific confirmation**.

### Follow-up: What did you learn from reading an established open-source codebase?

**Answer:** > **Needs candidate-specific confirmation** (e.g., Learned how they structure large dependency graphs, how they handle mock testing, or how CI/CD validates PRs).

---

## SECTION 2: TECHNOLOGY DEFENSE

### Q. Async/Await (Python)
**Basic:** What is asyncio in Python?
**Answer:** It's a library to write concurrent code using the async/await syntax, utilizing an event loop to cooperatively switch contexts during I/O-bound operations.

### Follow-up: Project-specific: How did you use it in the sidecar?
**Answer:** > **Needs candidate-specific confirmation**. (e.g., Used FastAPI to accept requests concurrently without blocking the server on slow downstream ML evaluations).

### Follow-up: Mechanism: How does the event loop actually work?
**Answer:** The event loop runs in a single thread. When a coroutine hits an `await` on an I/O operation (like network access), it yields control. The event loop then suspends that task and picks up another task that is ready to run, maximizing CPU utilization during wait times.

### Follow-up: Tradeoffs: When should you NOT use asyncio?
**Answer:** You should never use it for CPU-bound tasks. Because asyncio is single-threaded, a heavy mathematical computation will completely block the event loop, starving all other async tasks of execution time.

### Follow-up: Failure: What happens if a coroutine raises an unhandled exception?
**Answer:** The individual task crashes. If nothing is awaiting that task, the exception is swallowed, and the event loop continues, possibly logging an error. This can lead to silent failures if background tasks aren't monitored.

### Follow-up: Debugging: How do you debug a blocked event loop?
**Answer:** Enable asyncio debug mode (`PYTHONASYNCIODEBUG=1`). It automatically logs warnings for coroutines that take too long to execute, helping pinpoint blocking synchronous calls mistakenly placed in async functions.

### Follow-up: Scaling: How do you scale an asyncio application?
**Answer:** Because a single event loop is bound to a single CPU core, you scale horizontally by running multiple worker processes (e.g., Gunicorn with Uvicorn workers) to utilize multiple cores.

---

## SECTION 3: OWNERSHIP (CRITICAL)

## Q. "You only contributed one PR. How significant was this really?"
**Answer:** While it was a single PR, it was a structural architectural shift. I didn't fix a localized bug; I refactored a blocking synchronous system into an asynchronous pipeline. This required understanding their core execution router, managing state transitions, and proving stability with 26 integration tests.

### Follow-up: What exactly did you write vs what already existed?
**Answer:** > **Needs candidate-specific confirmation**. (Identify exactly which files/modules were created vs modified).

### Follow-up: How long did this take from finding the issue to merge?
**Answer:** > **Needs candidate-specific confirmation**.

### Follow-up: Why this particular issue?
**Answer:** > **Needs candidate-specific confirmation**.

---

## SECTION 4: ATTACK MODE

## Q. "Explain exactly how the async pipeline prevents the blocking problem. If the user still has to wait for the policy, isn't it still blocking?"
**Answer:** It blocks the *user's specific request flow*, but it does not block the *server's thread pool*. In the synchronous model, the server thread waits idly for the HTTP response. In the async model, the server thread hands off the job and immediately returns to the event loop to serve *other users*. System throughput and concurrency scale, even if individual request latency remains similar.

## Q. "What would happen if you had to maintain this long-term?"
**Answer:** I would implement robust queue telemetry. We'd need monitoring on queue depth to autoscale the sidecar workers, dead-letter queues for unprocessable evaluations, and circuit breakers in the main app to fail-fast if the sidecar becomes unresponsive.

## Q. "26 tests — did you write all of them or were some pre-existing?"
**Answer:** > **Needs candidate-specific confirmation**.

---

# PART 2: Research Intern — NIT Jamshedpur & IIT Guwahati

## SECTION 1: BULLET INTERROGATION

## Q. Tell me about your research internship.
**Answer:** I worked as a research intern building simulation pipelines for autonomous underwater vehicles. Using the ArduPilot ecosystem, I created over 20 Python and C++ environments to safely test computer vision and autonomy algorithms before physical deployment.

### Follow-up: What is ArduPilot?
**Answer:** ArduPilot is a major open-source autonomous vehicle software platform. It provides flight control, navigation, and mission planning software for various vehicles, including drones, rovers, and submarines via ArduSub.

### Follow-up: What is an ROV? What is an AUV? What is the difference?
**Answer:** An ROV (Remotely Operated Vehicle) is tethered and controlled by a human operator on the surface. An AUV (Autonomous Underwater Vehicle) is untethered and operates entirely autonomously based on onboard software and sensors.

### Follow-up: What exactly is a "simulation pipeline"?
**Answer:** It is an automated sequence that launches a physics environment, spawns a virtual vehicle, connects it to the software flight controller, feeds it a test mission, and logs the telemetry data to verify the vehicle's behavior without human intervention.

### Follow-up: Why simulate instead of testing on real hardware?
**Answer:** Hardware iteration is slow, expensive, and risky. Underwater testing requires massive logistical overhead like pools or boats, and a software bug can result in losing the hardware entirely. Simulation allows rapid, parallel, and safe testing of edge cases.

### Follow-up: What specific simulation environment did you use?
**Answer:** > **Needs candidate-specific confirmation** (Usually SITL - Software In The Loop, often paired with Gazebo, Webots, or BlueOS environments for physics and graphics rendering).

### Follow-up: Why use both Python AND C++? What was each used for?
**Answer:** Python was used for high-level scripting, orchestrating the pipeline execution, and data analysis. C++ was required for low-level tasks, such as modifying the core ArduPilot flight controller firmware or writing high-performance plugins for the physics simulator.

### Follow-up: What counts as one "pipeline"? Why did you need 20+?
**Answer:** A single pipeline represents a specific, isolated test scenario—for example, "navigate through three waypoints under high simulated current," or "track a visual marker while experiencing sensor noise." Building 20+ allowed us to cover a wide matrix of autonomy behaviors and environmental edge cases.

## Q. What OpenCV techniques were used in the vision modules?
**Answer:** > **Needs candidate-specific confirmation** (e.g., color thresholding, feature matching, optical flow, or ArUco marker detection for underwater docking).

### Follow-up: Why is underwater vision particularly challenging?
**Answer:** Water heavily attenuates light, causing severe color distortion by rapidly absorbing reds. Turbidity (suspended particles) scatters light, drastically reducing contrast. Lighting conditions also change unpredictably based on depth and surface weather.

### Follow-up: How does a Python OpenCV script actually control the ArduPilot system?
**Answer:** The script processes the camera feed, calculates positional offsets or velocities, and packages this data into MAVLink messages (like `VISION_POSITION_ESTIMATE`). This is sent over a UDP connection to the flight controller, which fuses it into its EKF (Extended Kalman Filter) to adjust actuators.

### Follow-up: What is MAVLink?
**Answer:** MAVLink is a lightweight, binary telemetry and command protocol used extensively in robotics to communicate efficiently between ground control stations, companion computers, and flight controllers.

### Follow-up: How did you handle turbidity and variable lighting in OpenCV?
**Answer:** > **Needs candidate-specific confirmation** (e.g., used histogram equalization, CLAHE, or dynamic thresholding algorithms to normalize the image before feature extraction).

## Q. What does "validated simulation outputs" mean in this context?
**Answer:** Validation means proving that the software's behavior in the simulation mathematically and observationally matches how it behaves in the real world, proving that the simulation tests are actually trustworthy.

### Follow-up: What was the validation methodology?
**Answer:** We execute a specific mission in the simulation, record the telemetry logs (position, velocity, actuator outputs), then execute the exact same mission on physical hardware in a pool, and statistically compare the two datasets to measure the variance.

### Follow-up: How do you fix an inaccurate simulation?
**Answer:** You tune the physics engine parameters. You adjust the simulated mass, drag coefficients, center of buoyancy, and sensor noise models until the simulated vehicle's response curves tightly match the physical vehicle's response curves.

### Follow-up: What is the sim-to-real gap?
**Answer:** It is the inherent discrepancy between simulated performance and physical performance. It's caused by unmodeled complex physics, sensor noise that isn't perfectly Gaussian, and chaotic environmental factors like micro-currents that simulators simplify.

---

## SECTION 2: RESEARCH -> ENGINEERING BRIDGE

## Q. "You did research but you're applying for engineering. How are they different?"
**Answer:** Research prioritizes discovery, proof-of-concept, and academic publishing. Engineering prioritizes reliability, maintainability, scaling, and production impact. While my title was research, my daily tasks were heavy engineering: building automated CI pipelines, writing robust C++, and architecting complex test frameworks.

### Follow-up: "What specific engineering skills did you develop during research?"
**Answer:** Complex systems integration (connecting asynchronous vision systems to real-time flight controllers), rigorous automated testing (building the 20+ pipelines), and low-level system debugging (C++ memory management, network protocol debugging).

### Follow-up: "Why did you leave research for software engineering?"
**Answer:** I realized my passion lies in the implementation rather than the theory. I found more satisfaction in building the tools, optimizing the architecture, and writing clean, robust code than in the academic process of writing papers and theoretical analysis.

---

## SECTION 3: OWNERSHIP

## Q. What was YOUR specific contribution vs the team/advisor?
**Answer:** > **Needs candidate-specific confirmation**.

### Follow-up: Were any papers published based on this work?
**Answer:** > **Needs candidate-specific confirmation**.

### Follow-up: What was the most challenging pipeline you built?
**Answer:** > **Needs candidate-specific confirmation**.

### Follow-up: What was the hardest bug you had to track down?
**Answer:** > **Needs candidate-specific confirmation** (Often involves race conditions in simulation synchronization, tracking down segfaults in C++ plugins, or MAVLink packet loss).

---

## SECTION 4: ATTACK MODE

## Q. "20+ pipelines sounds like a lot of repetitive work. Define what a pipeline is structurally."
**Answer:** A pipeline was an automated orchestration script. It utilized Docker to spin up an isolated Gazebo instance, deploy the compiled ArduSub firmware, execute a predefined MAVLink mission script, log the output telemetry, and tear down the environment cleanly. 20+ represents the matrix of unique mission parameter configurations and edge-case scenarios we automated.

## Q. "Was this mostly running someone else's code or writing your own?"
**Answer:** > **Needs candidate-specific confirmation** (Candidate must explicitly differentiate between configuring existing ArduPilot code vs writing novel OpenCV integration code or custom C++ physics plugins).

## Q. "What C++ code did you personally write?"
**Answer:** > **Needs candidate-specific confirmation**.
