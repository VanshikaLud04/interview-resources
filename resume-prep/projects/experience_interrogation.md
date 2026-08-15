# Experience Defense — Evidence-Backed Version

> This file deliberately separates verified evidence from future design ideas. Never describe a design extension as something you shipped.

## Part 1 — Cordum Open Source Contribution

### First, correct the public record

Your public PR is [#263](https://github.com/cordum-io/cordum-packs/pull/263), authored by `VanshikaLud04`. It was **closed, not merged**. It added 142 lines across two files: a LangChain governance callback and its package export. The resume link to #264 should not be described as your merged PR: #264 was authored by another contributor and explicitly says your #263 “sparked the design rethink.”

### Q. What did you actually build in PR #263?

**Answer:** “I added a `govern()` helper and a `CordumGovernanceCallback` for LangChain. The helper attaches the callback through `Runnable.with_config()` for LCEL-compatible runnables, with a legacy `agent.callbacks` fallback. Before a tool starts, the callback sends its capability and risk tags to Cordum’s `/api/v1/policy/evaluate` endpoint. A deny or human-approval decision raises `PermissionError` before the tool runs.”

### Q. How did it communicate with Cordum?

**Answer:** “Over HTTP using `httpx`, to `/api/v1/policy/evaluate`. The request includes a topic, tenant, capability (the tool name), risk tags, JSON content type, and optionally an API key. The implementation has both synchronous `httpx.post()` and asynchronous `httpx.AsyncClient.post()` paths.”

### Q. Was it an asynchronous job pipeline or sidecar?

**Answer:** “No—the public PR I authored was a callback-based policy adapter, not a deployed sidecar or job pipeline. It performs a pre-execution HTTP policy evaluation. The later merged #264 took a different job-based design; I can discuss it as a design that was inspired by my PR, but I do not claim I implemented it.”

### Q. What happens on denial or human approval?

**Answer:** “The callback reads the policy decision. `DECISION_TYPE_DENY` and `DECISION_TYPE_REQUIRE_HUMAN` each raise `PermissionError` with the policy reason; therefore the governed tool does not proceed. I do not claim a deterministic HTTP 403 response because that is not what #263 implements.”

### Q. What did you write versus what was already there?

**Answer:** “The PR added `integrations/agent-adapters/cordum_agent_adapters/govern.py` and exported `govern` plus `CordumGovernanceCallback` in `__init__.py`. It built on the project’s existing adapter package and LangChain dependency.”

### Q. Was the PR merged? What review feedback did you receive?

**Answer:** “No. PR #263 was closed. The public record shows that the follow-up merged PR credited it as the spark for a design rethink. I do not invent private review comments or say that I merged 26 tests.”

### Q. What did you learn from it?

**Answer:** “I learned how a framework integration should preserve its host framework’s execution model: use LangChain callbacks, support both LCEL and legacy callback attachment, keep policy metadata explicit, and make the deny/approval path fail before a tool executes.”

### Q. How did you use asyncio in this work?

**Answer:** “The callback exposes an `on_tool_start_async` method and makes the policy call with `httpx.AsyncClient`. The synchronous method remains for normal callback execution. I do not claim FastAPI, a queue, or worker deployment for this PR.”

### Q. Why is this still meaningful if it was not merged?

**Answer:** “It was a concrete framework integration with a clear design: policy checks before tool execution, LCEL compatibility, and typed handling of deny/approval outcomes. The useful lesson was responding to a design change rather than overstating ownership of the later implementation.”

## Part 2 — Research Internship

### Evidence boundary

The resume establishes the internship dates, the 20+ Python/C++ ArduSub/ArduPilot simulation pipelines, OpenCV-based underwater feature tracking, and iterative validation with researchers. The source code, simulation logs, papers, and personal work diary are not in this study repository. The answers below therefore stay at that evidence level; replace them only with details you can open and explain.

### Q. Tell me about the internship.

**Answer:** “I built 20+ Python/C++ simulation pipelines for ArduSub ROV/AUV testing in the ArduPilot ecosystem. I integrated OpenCV-based vision modules for underwater feature tracking and iteratively validated simulation outputs with researchers. It taught me how to turn a robotics experiment into repeatable engineering workflows.”

### Q. What was the exact simulator or environment?

**Answer:** “I only name the simulator after checking my project files or notes. ‘ArduPilot ecosystem’ and ‘ArduSub ROV/AUV testing’ are supported by my resume; Gazebo, SITL, Webots, BlueOS, or any other tool is not safe to claim from this material alone.”

### Q. Which OpenCV technique did you use for underwater tracking?

**Answer:** “I describe only the method I can locate in my code—not a plausible list of OpenCV techniques. The safe starting point is that underwater feature tracking was integrated; an interviewer may ask about contrast loss, color absorption, turbidity, and lighting variation, but I do not say I solved them with CLAHE, ArUco, or optical flow without evidence.”

### Q. How did you validate simulation outputs?

**Answer:** “The resume supports iterative validation with researchers. I should state the actual comparison or acceptance criterion from my notes; I do not claim pool-hardware comparison, calibrated physics parameters, or statistical variance analysis unless I have the corresponding experiment records.”

### Q. What was your contribution versus the advisor/team?

**Answer:** “My defensible ownership claim is the development of 20+ Python/C++ simulation pipelines and the OpenCV integration. I should name only the modules I wrote or modified, and clearly distinguish those from ArduPilot/ArduSub and simulator components supplied by the ecosystem.”

### Q. Were papers published? What was the hardest bug or pipeline?

**Answer:** “These are personal-history questions, not questions that a model should invent. I answer from my project records; if there was no paper, I say so. If I cannot name a bug, I use a smaller, real debugging example rather than fabricating a race condition, a C++ segfault, or MAVLink packet loss.”

### Q. Where did you use C++?

**Answer:** “The resume supports Python/C++ simulation pipelines. Before an interview, I will open the exact C++ file or commit I authored and prepare its purpose, inputs/outputs, and one trade-off. Until then, I do not claim firmware changes, custom physics plugins, or memory-management work.”

## Non-negotiable interview rule

If a detail is not supported by your code, public PR, experiment artifact, or contemporaneous notes, answer at the correct level of certainty. A narrow truthful answer is much stronger than an elaborate one that can be disproved by the linked source.
