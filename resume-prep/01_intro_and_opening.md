# Intro & Opening — Q&A

---

## ✅ Final Pitches (v3 — Red-Teamed)

> ⚠️ Confirm Locust config before using any specific number: check `locustfile.py` for `user_count`, `spawn_rate`, `test duration`, and total requests completed. If you can't recite those exactly, use "high concurrency" instead of "7,100."

### For Technical Interviews (Engineer-Facing) — 60 sec
> "Hi, I'm Vanshika, third-year EE at NIT Jamshedpur. For the past two years I've been building backend projects in the AI infrastructure space, mostly in Python.
>
> My strongest project is LLM Cost Guard — an LLM proxy that enforces token budgets across multiple providers. The core technical problem was concurrent budget over-authorization: two requests reading the same budget simultaneously, both passing the check, together exceeding the limit. I fixed it with an atomic Redis Lua script — the check and decrement happen in a single atomic operation in Redis. I validated it with a Locust load test at high concurrency — zero budget overruns across the full run.
>
> I've also built RAGOS — it automates configuration search for RAG pipelines. You define latency and cost constraints, it runs Random Search over 100+ parameter combinations, and returns a Pareto frontier. The constraint filtering before each trial is the interesting part — no wasted API calls on configurations that can't satisfy the budget.
>
> And Focus Lock — local computer vision using YOLOv8 and MediaPipe. I used Shannon entropy on frame differences to decide when to skip YOLO inference. During stationary desk sessions, this cut YOLO compute by roughly 93%.
>
> I'm applying because the problems — concurrent resource management, optimization under constraints, system reliability — map directly to what I've been building."

### For HR/Recruiter — 45 sec
> "Hi, I'm Vanshika, third-year EE at NIT Jamshedpur. I've spent two years building backend projects around AI — primarily the systems that control cost, reliability, and performance when using LLMs.
>
> My main project is LLM Cost Guard — an LLM proxy with token budget enforcement. I validated it under high concurrent load and got zero budget errors. I also built RAGOS, which automatically searches for optimal RAG pipeline configurations under latency and cost constraints, and Focus Lock — an on-device attention tracker that runs entirely locally using computer vision.
>
> I also contributed a merged PR to Cordum, an open-source LLM framework, where I redesigned their policy enforcement pipeline from synchronous to async.
>
> I'm interested in [Company] because [specific reason]."

### For Startup (Generalist) — 45 sec
> "Hi, I'm Vanshika, third-year EE at NIT Jamshedpur. I build backend systems around AI — token budget enforcement, RAG optimization, edge inference.
>
> The technically interesting one is LLM Cost Guard — I used an atomic Redis Lua script to close a TOCTOU race condition in token reservation under concurrent load. I also built a focus tracker with 97.2% recall running fully on-device.
>
> I want to work somewhere I own things end-to-end."

---

## What's Different from v1 and Why

| v1 Problem | v2 Fix |
|-----------|--------|
| "Intersection of AI and backend" as opener | Replaced with "infrastructure layer around LLMs" — specific, not buzzword |
| Cordum mentioned prominently | Moved to end or removed from lead — one PR is not your headline |
| "RAG optimization platform" tells nothing | Replaced with "AutoML for RAG — constraint-aware search, Pareto frontier" |
| "I love building infrastructure" | Deleted — nobody says this |
| No Focus Lock | Added — 97.2% recall + Shannon entropy is technically distinctive |
| "in production" claim | Removed — can't defend it |
| EE apologized for with "but" | Replaced with "my actual work has been" — degree becomes irrelevant |

---

## Common Opening Questions

## Q. "Why EE if you're interested in CS?"
**A:** "I don't see them as separate. EE gave me first-principles thinking about constraints and optimization — the same mathematical thinking I apply to backend systems. In Focus Lock I used Shannon entropy for adaptive CPU optimization, which is the exact same math from my signals courses. The transition wasn't a pivot, it was an extension."

## Q. "Walk me through your resume."
**A:** "Three buckets. First: **AI Infrastructure** — LLM Cost Guard handles concurrent token budgets with atomic Redis scripts; RAGOS automates RAG optimization under constraints. Second: **Open Source** — contributed an async policy pipeline to Cordum. Third: **Applied Research** — 10-month internship building CV simulation pipelines for underwater robots at NIT Jamshedpur and IIT Guwahati."

## Q. "What are you passionate about?"
**A:** "Concurrency and performance. I love finding where systems break under load and fixing the root cause rather than patching around it. The Redis Lua script in LLM Cost Guard was that kind of problem — the naive solution (read-check-write) has a TOCTOU race condition that only shows up under high concurrency."

## Q. "Why this role?"
**A:** "The problems I've been solving — token rate limiting, async task queues, vector search, concurrent request handling — these are backend infrastructure problems. I want to solve them at a scale where the impact is measurable."

## Q. "Why [Company]?" (Template)
**A:** "Two specific things: first, [engineering blog post / specific product decision] — that's the kind of tradeoff I find interesting. Second, [specific engineering challenge they're known for]. I've been dealing with a smaller version of that problem in [your project]."

## Q. "Biggest weakness?"
**A:** "I sometimes spend too long understanding internals before building. For RAGOS I read HNSW papers for three days before I realized I should just implement ChromaDB and understand it through usage. I'm actively balancing depth with shipping speed."

## Q. "Where in 5 years?"
**A:** "Senior backend engineer or systems architect. The person who's trusted to design distributed systems that actually hold up under load, and who helps junior engineers avoid the mistakes I made."

## Q. "What makes you different?"
**A:** "Most people who work with LLMs treat them as a black box API. I've built the infrastructure around them — the rate limiting, the routing, the cost tracking, the optimization layer. I think about LLMs as a systems engineering problem."

## Q. "Defend your CGPA (7.57)."
**A:** "Deliberate tradeoff. I was running 30+ hours/week of engineering work while maintaining a respectable CGPA in a rigorous EE curriculum. My 26/26 integration tests passing at Cordum, 7,100 concurrent tests in LLM Cost Guard — that's the engineering quality signal that matters for this role."

## Q. "No corporate internship?"
**A:** "I chose for depth over brand name. A 10-month research internship across two institutions where I built 20+ simulation pipelines from scratch, plus open-source contribution with a real PR review cycle — I got more architectural ownership than a typical summer internship."

---

## Company Research — Before Every Interview
1. **Core Product** — use it if possible
2. **Tech Stack** — JD, GitHub org, engineering blog, StackShare
3. **Recent Engineering Challenge** — search "[Company] engineering blog"
4. **Competitors** — Crunchbase

**3 Questions to Always Ask:**
1. "What's the hardest technical problem this team is actively solving?"
2. "How do you handle tech debt vs new features?"
3. "I saw [specific thing from their blog] — how did that change things for this team?"

---

## Opening Traps

**"How's your day?"** → "Great — I was working through a concurrency problem this morning so my brain is warmed up." (Not "studying for exams.")

**"Any questions before we start?"** → "Will we focus on architecture/design or implementation/coding?" (Shows you're ready for either.)

**"You're rambling"** → Stop. Pause. "Sorry, let me answer your actual question." Self-awareness reads as maturity.

**HR vs Engineer** → HR: impact and metrics. Engineer: mechanism and implementation.
