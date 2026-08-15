## SECTION 1: BULLET INTERROGATION TREES

## Q. You mentioned developing a "privacy-first Edge AI analytics platform" with 97.2% recall. What exactly is this 97.2% recall measuring?

**Answer:** It measures the system's ability to successfully detect actual distraction events. Out of all the true distraction events that occurred (like looking away at a phone or leaving the frame), the system correctly flagged 97.2% of them. Recall is True Positives divided by (True Positives plus False Negatives). We prioritized recall over precision because missing a distraction event entirely defeats the purpose of a focus monitor, whereas a false positive (flagging distraction when they were just stretching) can be quickly dismissed by the user.

### Follow-up: How did you measure this? Did you use a standard dataset or build your own?

**Answer:** The repository includes `benchmarks/eval_metrics.py`, which computes per-class precision, recall, and F1 from ground-truth and prediction JSON files. However, the public repository does not include those input files or an annotation protocol. I can describe the script, but I should not claim a custom recording/annotation process unless I retain the underlying data and can show it.

#### Follow-up: Manual annotation is prone to error and small sample sizes. How many frames or hours of video did you actually evaluate to get this 97.2% figure?

**Answer:** I do not have a public raw dataset or reproducible frame/hour count in the repository. The README states 9,300+ labelled frames, but until the ground-truth and prediction artifacts are preserved and rerunnable, I treat the 97.2% number as a resume claim needing evidence—not as an academic benchmark I can elaborate with invented hours or split details.

##### Follow-up: What was your precision on that same dataset? And your F1 score?

**Answer:** The README records `DISTRACTED` precision `0.482`, recall `0.972`, and F1 `0.645`; those are the only project-specific figures documented. They show a high-recall, low-precision trade-off. Because the raw inputs are absent, I should not present the values as independently reproduced, and I must not replace them with the unsupported 89% / 93% figures.

###### Follow-up: You used YOLOv8 and MediaPipe. Which model handled the 97.2% recall?

**Answer:** It's a combined pipeline. YOLOv8 handles object detection (specifically the `cell phone` and `person` classes), while MediaPipe handles face mesh and head pose estimation. The 97.2% recall is the system-level metric output by the `FocusFSM`, meaning the state machine correctly transitioned to `DISTRACTED` 97.2% of the time a true distraction occurred.

####### Follow-up: How does YOLOv8 work fundamentally differently from earlier YOLO versions?

**Answer:** YOLOv8 is an anchor-free detector. Unlike YOLOv4 or v5 which relied on predefined anchor boxes (priors) to predict bounding boxes, YOLOv8 directly predicts the center of an object and its height/width. It uses a modified CSPDarknet backbone and a decoupled head—processing objectness, classification, and regression in separate branches. This anchor-free approach reduces the number of box predictions, speeding up NMS (Non-Maximum Suppression) and making inference faster.

######## Follow-up: What is NMS and why is it needed if it's anchor-free?

**Answer:** Non-Maximum Suppression (NMS) is still required because the network still predicts multiple bounding boxes around the same object, especially from adjacent grid cells that all think they contain the object's center. NMS filters these out by keeping the box with the highest confidence score and discarding any other boxes that have an Intersection over Union (IoU) with it above a certain threshold.

######### Follow-up: What YOLOv8 variant did you use, and how did you hit <30ms latency?

**Answer:** The checked-in configuration specifies `yolov8n.pt` at confidence `0.45`. The code chooses an available device automatically; it does not show an ONNX/OpenVINO export path. The public latency profiler currently uses a `time.sleep(0.01)` stub rather than real model inference, so I can explain the intended architecture but cannot use that script as proof of <30ms performance.

########## Follow-up: 30ms is just YOLO. MediaPipe face mesh also takes time. What was the total per-frame pipeline latency?

**Answer:** No end-to-end per-frame latency breakdown is captured in the public benchmark artifacts. The sound answer is architectural: a sampled frame goes through YOLO, MediaPipe/gaze, scoring, FSM, and non-blocking event publication; analytics and SQLite persistence are downstream. I should measure and report each stage before quoting a total.

########### Follow-up: What hardware exactly was this evaluated on?

**Answer:** The README labels the empirical table “Apple M2,” while the checked-in code does not record a machine model. I should only name the exact machine from the benchmark run once I have its recorded environment; I do not substitute an M1 Pro or imply ONNXRuntime evidence that is not in the repository.

############ Follow-up: What if the user is running an older Intel Mac? Does the <30ms latency hold up?

**Answer:** I have no checked-in Intel-Mac benchmark, so I do not quote a latency range. The design should reduce inference frequency on static scenes regardless of hardware, but hardware-specific performance requires a separate measured run.

## Q. Your third bullet mentions applying Shannon entropy to dynamically adapt inference frames and slash CPU consumption by 93%. What is Shannon entropy?

**Answer:** In information theory, Shannon entropy measures the average level of "information," "surprise," or "uncertainty" inherent in a variable's possible outcomes. The formula is H = -Σ p(x) log₂ p(x). In the context of computer vision and this project, I computed the entropy of the pixel intensity distribution (the histogram) of a frame or the difference between consecutive frames. 

### Follow-up: How exactly do you compute this on a video frame in code?

**Answer:** In `detection/sampler.py`, I convert the frame to grayscale. Then, I compute the absolute difference between the current frame and the previous frame. I calculate the histogram of this difference image, normalize it to create a probability distribution (so the sum of all bins equals 1), and then apply the Shannon entropy formula: multiplying each non-zero probability by its base-2 logarithm, summing them, and negating the result.

#### Follow-up: Why calculate entropy on the *difference* image instead of the frame itself?

**Answer:** If I calculated entropy on the frame itself, a highly textured background (like a bookshelf) would yield high entropy even if there is zero motion. By calculating entropy on the *difference* image, static backgrounds subtract out to zero. The resulting probability distribution reflects only what has changed. High entropy on the difference image means significant, complex motion occurred (like a person moving their hands or head). Low entropy means the frame is nearly identical to the previous one.

##### Follow-up: Why use entropy? Why not just use a simple pixel differencing threshold or optical flow?

**Answer:** Simple pixel differencing (e.g., counting pixels above a threshold) is highly sensitive to noise and global lighting shifts (like a cloud passing outside a window). Optical flow (like Lucas-Kanade) is robust but computationally expensive, defeating the purpose of saving CPU cycles. Entropy is a sweet spot: it evaluates the *complexity* of the difference. A global lighting shift changes all pixels uniformly, resulting in a low-entropy difference. Localized, complex human motion results in a high-entropy difference. 

###### Follow-up: What was the threshold you used for entropy, and how did you tune it?

**Answer:** The sampler has no fixed entropy threshold. It maps entropy continuously to an interval between `0.2s` and `3.0s`, then smooths that interval with an EMA of `0.25`. High entropy shortens the interval; low entropy lengthens it. The checked-in latency profiler does not log entropy values, so I do not invent bit thresholds.

####### Follow-up: You claim a 93% CPU reduction. What was the baseline and what exactly are you measuring?

**Answer:** The baseline was running the full inference pipeline (YOLOv8 + MediaPipe) blindly on every single frame at 30 FPS. The measurement was the total CPU time consumed by the Python process over a 10-minute active working session. By using the entropy sampler, the system skipped inference on roughly 93% of the frames (because during deep work, a person is largely static). It only fired the heavy neural networks during significant motion events, reducing peak CPU spikes and drastically lowering average CPU utilization.

######## Follow-up: Wait, if you skip frames when there's no motion, how do you handle a user who is frozen in a distracted state (e.g. staring motionless at a phone)?

**Answer:** This is a critical edge case. The FSM handles this by holding the last known state. If the last frame processed (before motion stopped and entropy dropped below threshold) was classified as DISTRACTED, the system continues to emit DISTRACTED events for the skipped frames. It assumes the user hasn't moved. If a time-to-live (TTL) of 5 seconds passes with no high-entropy frames, the sampler forces a keyframe through YOLO/MediaPipe just to double-check the state.

######### Follow-up: What profiling tool did you use to measure the 93%?

**Answer:** The public repository does not contain a `cProfile`/SnakeViz artifact or a real inference profiler that establishes the 93% figure. The sampler code does establish the mechanism for reducing inference frequency. Before defending a 93% reduction, I need a reproducible before/after run with the exact measurement method and raw outputs.


## SECTION 2: COMPUTER VISION DEEP DIVE

## Q. Walk me through the detection-to-decision pipeline. From the camera lens to the FSM transitioning to DISTRACTED.

**Answer:** 
1. **Capture:** `camera.py` pulls a BGR frame via OpenCV in a dedicated thread.
2. **Sampling:** `sampler.py` computes the entropy of the frame difference. If below threshold, skip.
3. **Inference:** The frame passes to `yolo_detector.py` to find bounding boxes for "person" and "cell phone", and to `gaze.py` (running MediaPipe) to extract the 468 face mesh landmarks.
4. **Scoring:** `focus_score.py` calculates pitch/yaw/roll from the face mesh to determine gaze direction. It combines this with YOLO's phone detection to generate a continuous focus score (0 to 100).
5. **State Machine:** `focus_fsm.py` reads this score. If the score drops below a threshold for N consecutive frames (debounce), it transitions from FOCUSED to DISTRACTED.
6. **Events:** The FSM emits a state-change event via `events.py` (EventBus), which the HUD overlays and Analytics engine consume.

### Follow-up: How do you extract head pose (pitch, yaw, roll) from a 2D face mesh?

**Answer:** I take 3D landmarks provided by MediaPipe (specifically the nose tip, chin, eye corners, and mouth corners). I define a generic 3D human face model (canonical face). Using OpenCV's `solvePnP` (Perspective-n-Point) algorithm, alongside the camera matrix (assumed or calibrated focal length), I compute the rotation and translation vectors that map the canonical 3D model to the 2D landmarks detected on the screen. The rotation vector is then converted to Euler angles (pitch, yaw, roll) using Rodrigues' rotation formula.

#### Follow-up: What happens to `solvePnP` if the user turns their head 90 degrees and half the landmarks are occluded?

**Answer:** `solvePnP` will fail or return highly unstable rotation vectors because MediaPipe's face mesh degrades significantly in profile views. To handle this, if MediaPipe fails to return landmarks, or if the confidence drops, the pipeline defaults to YOLOv8. If YOLO sees a "person" bounding box but no face is detected by MediaPipe for a sustained period, the system assumes the user has looked away completely, and the FSM penalizes the focus score.

##### Follow-up: How do you combine the YOLO detection and the head pose into a single score? 

**Answer:** In `focus_score.py`, it's a weighted penalty system. Base score is 100. If yaw/pitch exceed a "safe" threshold (e.g., looking down more than 20 degrees), subtract 30 points. If YOLO detects a "cell phone" intersecting with the person's bounding box, subtract 60 points. If no face is detected at all, subtract 80 points. The final score is clamped between 0 and 100.

###### Follow-up: Wait, if the user holds a phone but is looking at the screen, does it penalize them?

**Answer:** Yes, but the penalty depends on overlap. If YOLO detects a phone, we check if the user is looking at it based on the gaze vectors. If the phone is just resting on the desk (detected by YOLO but outside the interaction zone), the penalty is ignored. We do geometry intersection in `geometry.py` to ensure the phone is in the user's hand/gaze path before applying the severe penalty.

####### Follow-up: What exactly happens in `geometry.py` to determine "interaction zone"?

**Answer:** `geometry.py` computes the bounding box intersection between the user's detected hand/body region and the phone's bounding box. I cast a vector from the estimated eye centers along the calculated pitch/yaw trajectory. If this gaze vector intersects the YOLO bounding box for the phone, the user is confirmed to be looking at the phone, and the full 60-point penalty is applied.

######## Follow-up: How do you handle false positives where a rectangular object (like a calculator or wallet) is detected as a phone?

**Answer:** YOLO is notoriously bad at distinguishing black rectangles. I mitigate this by using the temporal nature of the video stream. The FSM applies a smoothing function. If a phone is detected for 2 frames but vanishes for 10, it's treated as a hallucination. If a phone is detected consistently, the confidence score threshold ensures only high-probability detections trigger the penalty. 


## SECTION 3: EVENT-DRIVEN ARCHITECTURE DEEP DIVE

## Q. You mentioned an in-memory EventBus separating capture, inference, telemetry, and analytics. How did you implement this?

**Answer:** I implemented the Publisher-Subscriber (Pub-Sub) pattern. In `events.py`, the EventBus is a singleton-like manager. It maintains a dictionary mapping EventTypes (e.g., `FocusStateChanged`, `TelemetryTick`) to a list of subscriber callback functions. When the `FocusFSM` transitions, it calls `bus.publish(FocusStateChanged(state="DISTRACTED"))`. The bus iterates through all registered callbacks (like the Dashboard WebSocket broadcaster, the SQLite writer, and the Analytics engine) and executes them.

### Follow-up: Are these callbacks executed synchronously on the thread that calls `publish()`?

**Answer:** No. If they were synchronous, writing to SQLite or sending over WebSockets would block the critical inference thread, ruining our <30ms latency. The EventBus uses Python's `queue.Queue` for subscribers. When `publish()` is called, the bus places the event into the queue of each subscriber. Each subscriber (like the Database writer) runs in its own thread, reading from its queue via a blocking `get()` loop.

#### Follow-up: If the SQLite writer thread gets slow, doesn't its queue grow infinitely and cause an OOM?

**Answer:** To prevent backpressure and OOM, the queues are bounded with a `maxsize`. If a queue is full, the `publish` method uses a non-blocking `put_nowait()`. If it raises a `queue.Full` exception, the bus drops the event for that specific subscriber and logs a warning. For analytics telemetry, dropping a frame is acceptable. For state changes, the queue size is large enough to handle bursts, and state changes are rare enough that the queue practically never fills.

##### Follow-up: Why not use `asyncio` instead of threading for this pipeline?

**Answer:** `asyncio` is great for I/O-bound tasks, but computer vision is highly CPU-bound. OpenCV's `VideoCapture.read()` is blocking, and running ONNX/YOLO inference is a blocking C++ call. If I used `asyncio`, these operations would block the event loop, freezing the WebSockets and analytics entirely. Python's `threading` allows the OS to preempt threads. Furthermore, YOLO and OpenCV release the Global Interpreter Lock (GIL) during their C/C++ execution, allowing true parallelism for the Python threads managing the DB and WebSockets.

###### Follow-up: You mentioned lock-safe queues. Did you implement your own locks?

**Answer:** No, I used Python's built-in `queue.Queue`. It abstracts away the locking. Internally, it uses `threading.Condition` and `collections.deque` to guarantee thread-safe FIFO ordering for multi-producer, multi-consumer scenarios. Using native `queue.Queue` prevents race conditions without me having to manually manage `threading.Lock()` acquisitions, reducing the risk of deadlocks.

####### Follow-up: If multiple threads are reading from the same queue, how do you prevent them from stealing each other's events?

**Answer:** In the Publisher-Subscriber model I implemented, subscribers do not share a single queue. Each registered subscriber gets its *own* instance of `queue.Queue`. When the publisher calls `bus.publish()`, it iterates over all subscriber queues and pushes a copy (or reference) of the event object into each one. This way, the Analytics thread and the DB thread process the exact same events independently without race conditions.

######## Follow-up: Memory references across threads. If the inference thread modifies the event object after publishing, does it corrupt the DB thread's data?

**Answer:** Yes, that's a classic data race. To prevent it, the events published (like `AttentionEvent`) are implemented as frozen Python `dataclass` instances (using `@dataclass(frozen=True)`). They are immutable. Once created, they cannot be modified. The DB thread reads the exact same immutable state that was originally generated.


## SECTION 4: THREADING & CONCURRENCY (GIL DEFENSE)

## Q. You said OpenCV and YOLO release the GIL. Prove it. If they didn't, what would your CPU profile look like?

**Answer:** In CPython, the GIL prevents multiple native threads from executing Python bytecodes at once. However, native C extensions (like numpy, OpenCV, and ONNXRuntime) can explicitly release the GIL using the `Py_BEGIN_ALLOW_THREADS` macro before starting heavy computation, and reacquire it after. If they didn't release the GIL, my multi-threaded architecture would be bottlenecked to a single core's capacity. The CPU profile would show the Python process maxing out at 100% (one core), and my UI/WebSocket thread would stutter wildly every time a frame was processed. Because they do release it, my process can utilize 200-300% CPU (multiple cores) when active, and the WebSocket thread remains responsive.

### Follow-up: What happens if the Camera thread crashes? How does the Inference thread know to stop?

**Answer:** In `main.py`, I use a `threading.Event` called `shutdown_flag`. If the camera thread raises an exception (e.g., USB unplugged), it catches it, sets `shutdown_flag.set()`, and exits. The Inference thread's main loop condition is `while not shutdown_flag.is_set():`. Additionally, the queues implement a timeout (`queue.get(timeout=1.0)`). This prevents the Inference thread from deadlocking indefinitely waiting for a frame that will never arrive. When the timeout hits, it checks the shutdown flag and gracefully exits.

#### Follow-up: How do you handle graceful shutdown for the SQLite Writer thread to ensure no data loss?

**Answer:** When `shutdown_flag` is set, the main thread pushes a special Sentinel value (e.g., `None` or a `ShutdownEvent`) into the EventBus queues. The SQLite writer thread processes its queue until it pulls the Sentinel. Once it sees the Sentinel, it flushes any remaining batched queries to the database, closes the SQLite connection safely, and breaks its loop. The main thread uses `thread.join()` to wait for this flush to complete before exiting the process.

##### Follow-up: What if the OS forcefully kills the process (SIGKILL) before the sentinel is processed?

**Answer:** SIGKILL (kill -9) cannot be caught in Python, so the process terminates immediately. Data in the queues and uncommitted SQLite transactions are lost. To mitigate this, SQLite transactions are batched tightly (every 5 seconds), meaning the absolute maximum data loss is 5 seconds of telemetry. The SQLite file itself remains uncorrupted due to its atomic commit protocol and WAL mode.


## SECTION 5: DATABASE & ANALYTICS

## Q. Why use SQLite instead of a time-series database like InfluxDB or a standard Postgres DB?

**Answer:** Focus Lock is an Edge AI application designed to be privacy-first. It runs entirely locally on the user's Mac. Asking a user to spin up a Docker container with Postgres or InfluxDB is awful UX. SQLite is embedded, zero-configuration, and stores everything in a single local `.db` file. Since there's only one user (the local machine) writing to it, SQLite easily handles the write volume, and it requires no background service.

### Follow-up: How did you design the schema for storing continuous telemetry data?

**Answer:** `database.py` creates `sessions`, `events`, `attention_events`, `daily_summary`, and `weekly_summary`. `attention_events` records session, timestamp, state, trigger, confidence, active app, FPS, CPU/RAM, face/phone flags, and gaze angles. The database indexes `session_id` and timestamp columns for `events` and `attention_events`, and enables WAL mode on connections.

#### Follow-up: Writing to a database on a disk at 30 FPS will destroy performance. How did you optimize this?

**Answer:** I implemented batch writing in `sqlite_writer.py`. Instead of executing an `INSERT` statement for every single frame, the DB thread accumulates events in an in-memory list. Once the list hits 100 items, or a time threshold of 5 seconds passes, it executes an `executemany()` operation inside a single transaction (`BEGIN; INSERT...; COMMIT;`). This reduces disk I/O operations by two orders of magnitude and avoids SQLite's file-locking bottleneck.

##### Follow-up: What happens if the batch insert fails midway due to a schema error or locked database?

**Answer:** SQLite's atomic transactions ensure that if `executemany()` fails halfway, the entire batch is rolled back. The database is never left in a partially updated state. My python code catches the `sqlite3.Error`, logs it, and attempts an exponential backoff retry. If it fails 3 times, it drops the batch to prevent queue overflow and proceeds to the next block of events.


## SECTION 6: STATE MACHINE & HUD

## Q. Let's talk about the FocusFSM. You mentioned debouncing to avoid flickering between FOCUSED and DISTRACTED. How is that implemented mathematically?

**Answer:** In `fsm/focus_fsm.py`, the FSM uses a rolling buffer of the last $N$ focus scores. Instead of transitioning immediately when a single frame's score drops below the threshold (which could be a false positive from a blink or a bad detection), the FSM requires the *moving average* of the last $N$ frames to drop below the threshold. Alternatively, I used a counter: if the score is below threshold, increment counter. If above, decrement. Only transition state when the counter hits a limit (e.g., 30 frames / 1 second of continuous distraction). 

### Follow-up: What happens on macOS when the FSM officially transitions to DISTRACTED?

**Answer:** When the state hits DISTRACTED, the FSM publishes a `DistractionEvent`. The `macos/accessibility.py` and `hud/overlay.py` subscribers pick this up. Using `tkinter` or PyQt, a transparent, borderless, always-on-top window is spawned over the screen, darkening the edges and displaying a subtle "Focus Lost" warning. This serves as a psychological nudge.

#### Follow-up: How do you bypass macOS security to draw an always-on-top overlay?

**Answer:** By setting the window level to `NSStatusWindowLevel` or `NSScreenSaverWindowLevel` via macOS-specific API calls, or using standard UI framework flags like `w.attributes('-topmost', True)` in Tkinter. It doesn't bypass security; it just uses standard accessibility window floating. The app *does* require Screen Recording permissions to draw over other apps properly without being hidden by Spaces, and Camera permissions to access the feed.

##### Follow-up: How does the app request those permissions seamlessly from the user?

**Answer:** In `macos/accessibility.py`, before launching the camera loop, the application checks `AVCaptureDevice` authorizations using `pyobjc`. If permissions are missing, it triggers standard macOS dialog boxes requesting Camera and Screen Recording access, guiding the user to System Preferences. The application enters a waiting loop until the permissions are granted.


## SECTION 7: TECHNOLOGY DEFENSE (7 LEVELS)

### 1. Python Threading
*   **Basic:** Threads run concurrently within a single process.
*   **Project-specific:** Used to isolate camera capture, YOLO inference, DB writing, and Flask server.
*   **Mechanism:** Maps to OS-level threads (pthreads on macOS). Python's interpreter manages them subject to the GIL.
*   **Tradeoffs:** Easy memory sharing and IPC compared to multiprocessing. However, true multi-core utilization for pure Python code is impossible due to the GIL.
*   **Failure:** A thread encounters an unhandled exception and dies silently, leaving queues to fill up infinitely (OOM).
*   **Debugging:** `threading.enumerate()` to monitor live threads. `queue.qsize()` to detect bottlenecks. Use `faulthandler` to dump tracebacks on crash.
*   **Scaling:** Will not scale beyond a few specialized I/O bounds. If scaling to process 10 cameras simultaneously, Python threading will bottleneck heavily. Must migrate to `multiprocessing` or C++/Rust.

### 2. YOLOv8
*   **Basic:** A fast, real-time object detection CNN.
*   **Project-specific:** Locates the user's face and cell phone in real-time.
*   **Mechanism:** Uses an anchor-free architecture to predict bounding boxes directly, extracting features via CSPDarknet.
*   **Tradeoffs:** Incredibly fast and lightweight (Nano version), but sacrifices precision on extremely small objects compared to slower R-CNN models.
*   **Failure:** Hallucinates phones in shadows or fails to detect a phone if the user's hand entirely covers it.
*   **Debugging:** Extract the internal confidence tensors. Visualize bounding boxes using `cv2.rectangle()`. Evaluate against a labeled test set using standard COCO metrics (mAP).
*   **Scaling:** Export to OpenVINO, TensorRT, or CoreML to utilize specialized Neural Processing Units (NPUs) on target hardware.

### 3. MediaPipe Face Mesh
*   **Basic:** A high-fidelity facial geometry tracker.
*   **Project-specific:** Used for determining if the user's gaze is directed at the screen or away.
*   **Mechanism:** Uses machine learning to infer 468 3D landmarks in real-time from a 2D image, utilizing a two-step pipeline (face detection -> landmark regression).
*   **Tradeoffs:** Very accurate for head pose, but struggles with extreme angles (full profile view) where landmarks are occluded.
*   **Failure:** Fails to initialize if the user is backlit, or returns jittery coordinates if lighting is poor.
*   **Debugging:** Render the 468 landmarks over the frame. Apply a Kalman filter to smooth the coordinates if they jitter excessively.
*   **Scaling:** Can be configured to track multiple faces, but computational cost scales linearly with each face added.

### 4. SQLite
*   **Basic:** An embedded SQL database engine.
*   **Project-specific:** Stores the application's telemetry and focus sessions locally.
*   **Mechanism:** Stores all tables, indices, and data in a single cross-platform file. Executes SQL queries directly within the application process.
*   **Tradeoffs:** Perfect for local, zero-config apps. Terrible for highly concurrent write operations.
*   **Failure:** Throws `database is locked` exceptions if multiple threads attempt to write simultaneously or if a read transaction takes too long.
*   **Debugging:** Analyze `.db` file using DB Browser for SQLite. Use `EXPLAIN QUERY PLAN` to optimize the analytics queries.
*   **Scaling:** Enable Write-Ahead Logging (WAL). If the app were to sync to the cloud, SQLite would serve as the local offline cache syncing to a Postgres backend.

### 5. Flask & WebSockets
*   **Basic:** Flask is a micro web framework. WebSockets provide full-duplex communication.
*   **Project-specific:** Hosts a local HTML dashboard and pushes real-time focus states to the UI.
*   **Mechanism:** Flask-SocketIO upgrades the HTTP request to a WebSocket connection, keeping a TCP socket open for bidirectional JSON messaging.
*   **Tradeoffs:** Real-time updates without HTTP polling overhead, but requires complex connection management (reconnects, heartbeats).
*   **Failure:** The dashboard tab is put to sleep by macOS/Chrome, causing the WebSocket to sever and miss state updates.
*   **Debugging:** Check the browser's Network tab under WS. Look for ping/pong frames. Ensure the Flask server is emitting on the correct channel.
*   **Scaling:** If deployed to production, Flask-SocketIO requires a message broker (like Redis or RabbitMQ) to sync connections across multiple worker processes (Gunicorn).

### 6. Information Theory (Entropy)
*   **Basic:** A measure of uncertainty or complexity.
*   **Project-specific:** Determines if a frame contains enough novel motion to warrant running expensive inference.
*   **Mechanism:** Computes the distribution of pixel differences, applies $-p \log_2(p)$, and compares against a threshold.
*   **Tradeoffs:** Computationally cheap gatekeeper. However, highly sensitive to threshold tuning; too low wastes CPU, too high misses distractions.
*   **Failure:** A flickering desk lamp could artificially inflate entropy, forcing inference on every frame despite the user being still.
*   **Debugging:** Plot entropy values in real-time on a graph next to the video feed. Tune the threshold by analyzing the histogram of entropy values across different environments.
*   **Scaling:** Instead of global frame entropy, apply entropy calculation on a localized grid to isolate motion to specific zones (e.g., ignoring a window but watching the user).

### 7. EventBus Pattern
*   **Basic:** A software architectural pattern facilitating communication between decoupled components.
*   **Project-specific:** Routes data from the inference engine to the DB, Analytics, and UI without tightly coupling them.
*   **Mechanism:** Uses topic-based routing with python `queue.Queue` to deliver event objects asynchronously to subscriber threads.
*   **Tradeoffs:** Excellent separation of concerns. However, it obfuscates control flow, making the code harder to trace sequentially.
*   **Failure:** A slow subscriber causes its queue to back up until the application runs out of memory.
*   **Debugging:** Log the size of every subscriber queue on a timer. If one grows unbounded, the subscriber is blocked or dead.
*   **Scaling:** Migrate from in-memory queues to a distributed message broker like Kafka or RabbitMQ if components are split into microservices.


## SECTION 8: SECURITY, PRIVACY & ATTACK MODE

## Q. "You said Privacy-First Edge AI. But you're capturing video of users in their homes. How do I know this isn't malware?"

**Answer:** The system is air-gapped from the cloud by design. There is no cloud API key in the codebase. All models (YOLOv8, MediaPipe) run on the local CPU/GPU. The SQLite database is local. The `serve.py` Flask server only binds to `127.0.0.1` (localhost), meaning it cannot be accessed from the wider internet or even the local network. Code can be audited to prove no external outbound HTTP requests are made containing image data.

### Follow-up: What if I use Wireshark? Will I see any packets leaving the machine?

**Answer:** You will only see the WebSocket packets traversing `localhost` (127.0.0.1) between the Python backend and the local HTML dashboard. You will not see any outbound packets to external IPs.

#### Follow-up: "You claim <30ms inference latency. Prove it. What hardware did you test this on, and how did you profile it?"

**Answer:** The repo includes a latency-profile scaffold using `time.perf_counter`, but its actual model invocation is commented out and replaced with `time.sleep(0.01)`. Therefore it cannot substantiate <30ms or a CoreML/OpenVINO path. I should describe this as benchmark scaffolding and rerun it with real inference before making performance claims.


## SECTION 9: ARCHITECTURE CRITIQUE & FUTURE WORK

## Q. Looking back at the architecture, what is the weakest link? What would you change if you had 3 more months?

**Answer:** The weakest link is relying on RGB camera feed for head pose estimation in poor lighting. If the user works in a dark room, YOLO and MediaPipe degrade drastically, dropping recall. If I had 3 months, I would integrate infrared (IR) camera support (like Windows Hello/FaceID cameras) for illumination-invariant tracking. 
Additionally, I would replace Python threads with Python `multiprocessing` combined with Shared Memory (`multiprocessing.shared_memory`). Right now, passing large numpy arrays (frames) through `queue.Queue` involves serialization/deserialization or memory copying overhead between some boundaries. Using a shared memory buffer would allow the camera process to write the frame once, and the inference process to read it via a pointer, eliminating copy overhead entirely.

### Follow-up: If you move to multiprocessing, you lose the EventBus's ability to easily pass objects in memory. How do you fix that?

**Answer:** I would swap the in-memory `queue.Queue` EventBus for a lightweight IPC mechanism like Redis or ZeroMQ (ZMQ). ZMQ operates via sockets (like TCP or IPC) and is extremely fast. The inference process would publish JSON events over a ZMQ PUB socket, and the database/dashboard processes would listen on SUB sockets. This maintains the decoupled architecture while supporting multi-process scaling.

#### Follow-up: You mentioned quantization earlier. Why didn't you deploy an INT8 quantized YOLO model?

**Answer:** INT8 quantization is not implemented or benchmarked in the public repository. I should not claim an observed accuracy drop or a model-selection experiment. A defensible future-work answer is that I would benchmark FP32 and INT8 on the same labelled set, comparing recall, precision, and latency before deciding.

##### Follow-up: Explain how INT8 quantization actually causes that precision drop mathematically.

**Answer:** Neural networks are typically trained using FP32 (32-bit floating point), which has a massive dynamic range. Quantization maps this continuous FP32 range into 256 discrete integer values (INT8). This mapping involves a scale and zero-point. If a specific layer has outliers—activation values that are unusually high or low—the quantization algorithm must stretch the INT8 range to cover the outliers. This means the subtle variations between the "normal" values get crushed into the same integer bin. When those subtle variations are critical for detecting small features (like the edge of a phone case vs a hand), the model loses the ability to distinguish them, leading to false negatives and lower recall.

###### Follow-up: How do you mitigate that? Post-training quantization vs Quantization-Aware Training (QAT)?

**Answer:** Exactly. I used Post-Training Quantization (PTQ) because it was fast, but it caused the precision drop. To fix it, I would need to use Quantization-Aware Training (QAT). In QAT, you insert "fake quantization" nodes into the network during training/fine-tuning. This forces the model to learn weights that are robust to the precision loss before the actual INT8 conversion happens. It takes much more compute and time to train, which is why it wasn't implemented in this iteration. 

## SECTION 10: FAILURE SCENARIOS & RECOVERY

## Q. The application is running. The user suddenly unplugs their external webcam. What happens in the code?

**Answer:** OpenCV's `cap.read()` will start returning `(False, None)`. In `camera.py`, I check the boolean return value. If it's False, I increment a failure counter. If it fails for 5 consecutive frames, the Camera thread logs an error, sets the `shutdown_flag` event, and publishes a `CameraDisconnected` event to the EventBus.

### Follow-up: Does the UI freeze when this happens?

**Answer:** No. Because of the Event-Driven Architecture, the UI is decoupled. The Dashboard (via WebSocket) receives the `CameraDisconnected` event and immediately renders an error state to the user, prompting them to reconnect the camera. The inference thread sees the `shutdown_flag` and gracefully exits.

#### Follow-up: The user plugs the camera back in. How does the system recover?

**Answer:** Currently, the user would need to click a "Restart Session" button on the dashboard. This triggers an API call to `serve.py`, which re-initializes the `camera.py` thread and resets the FSM state. If I were to automate it, I would implement a watchdog thread that periodically attempts to re-open the OpenCV `VideoCapture` index when in a disconnected state.

## Q. What happens if the SQLite database gets locked because the Analytics thread is running a massive heatmap query while the Writer thread is trying to insert events?

**Answer:** SQLite uses file-level locking. If a long `SELECT` query runs, it can block `INSERT`s, causing `sqlite3.OperationalError: database is locked`. 

### Follow-up: How did you fix or prevent this?

**Answer:** I enabled Write-Ahead Logging (WAL) mode by executing `PRAGMA journal_mode=WAL;` on the database connection. WAL mode allows concurrent readers and writers. The reader (Analytics engine) reads from the main database file, while the writer appends to the WAL file. This completely eliminates the read/write lock contention. Furthermore, the batch writing in the Writer thread is wrapped in a `try...except` block with an exponential backoff retry mechanism just in case a lock does occur.

---
> **End of Interrogation Document.**

## SECTION 11: ADDITIONAL ATTACK VECTORS & EDGE CASES

## Q. What if the user places a printed photo of themselves in front of the camera?
**Answer:** This is a classic presentation attack (spoofing). MediaPipe face mesh will likely detect the face in the 2D photo and return landmarks. However, the system can mitigate this by checking for micro-movements (like blinking or slight head jitter) over time. If the landmarks remain perfectly static (entropy is exactly zero for an extended period), the system can flag it as a spoofing attempt.

### Follow-up: Did you implement this anti-spoofing mechanism?
**Answer:** No anti-spoofing or static-timeout mechanism is present in the public code. The honest answer is: “I did not implement liveness detection; a printed-photo attack is outside the project’s current threat model. A future version could add blink/temporal checks or a dedicated anti-spoof model.”

## Q. What if the user wears a mask or heavy sunglasses?
**Answer:** Heavy sunglasses occlude the eye landmarks. MediaPipe face mesh confidence for the eye regions will drop significantly.

### Follow-up: How does the system handle that?
**Answer:** The system falls back on global head pose. Even if the eyes are occluded, the nose, chin, and face outline usually provide enough 3D geometry for `solvePnP` to estimate the pitch and yaw. If the user turns their head away, the system will still detect it.

## Q. You mentioned OpenCV camera threads crashing. What happens if the OS puts the USB bus to sleep?
**Answer:** macOS aggressively power-manages USB ports. If the OS sleeps the bus, `cap.read()` will block or return False. 

### Follow-up: How did you prevent this?
**Answer:** I used a macOS-specific assertion (using `caffeinate` or `IOPMAssertionCreateWithName`) to prevent the system from sleeping the display or idle-sleeping while a focus session is active.

## Q. How do you handle multiple people in the camera frame?
**Answer:** YOLO will return multiple "person" bounding boxes, and MediaPipe can return multiple face meshes.

### Follow-up: Which person does the system track?
**Answer:** In `geometry.py`, I track the person with the largest bounding box area, assuming the user sitting at the computer is closest to the lens. The system explicitly ignores background detections.

#### Follow-up: What if someone walks behind the user and holds up a phone?
**Answer:** YOLO will detect the phone, but the `geometry.py` intersection logic checks if the phone bounding box overlaps with the *primary* user's bounding box and is within the primary user's gaze vector. Since the secondary person's phone is outside this zone, it is ignored.

