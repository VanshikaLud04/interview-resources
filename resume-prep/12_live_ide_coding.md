# Live IDE Coding Survival Guide

> This is not a LeetCode round. The interviewer is usually checking whether you can turn a small, ambiguous engineering task into readable, testable code. You do **not** need to know every library by memory. You need a calm process.

## First: What They May Actually Ask You to Build

For your Python/backend/AI profile, the realistic small tasks are usually one of these:

| Category | Typical prompt | What they are testing |
|---|---|---|
| Data transformation | "Parse these events and return usage per user." | clean loops, dictionaries, edge cases |
| Small API | "Add a validated endpoint / pagination / auth check." | function boundaries, validation, HTTP thinking |
| Async / integration | "Call two services concurrently with timeout/retry." | `async` basics, failure handling |
| Cache / rate-limit logic | "Implement a simple TTL cache / token bucket." | invariants, time handling, tests |
| Debugging | "This code is slow/wrong—fix it." | reading before writing, hypothesis-driven changes |
| Database | "Write the query/schema for this access pattern." | joins, indexes, transactions, clarity |
| Tests | "Write tests for this helper." | boundary cases and confidence |
| Small CLI / file task | "Read JSON/CSV, aggregate it, write output." | practical Python, error handling |

They may ask a basic C++ task too, but your strongest framing is: "I practise algorithms in C++, and for backend implementation I am most fluent in Python." That is a normal, honest answer.

---

## The First 90 Seconds: Do This Every Time

Say this before touching the keyboard:

> "I’ll restate the behaviour, clarify one edge case, sketch the smallest interface, then implement the happy path and test the boundaries."

Then follow this order:

1. **Restate input and output.** "We receive a list of events and return one total per organisation, correct?"
2. **Ask only material questions.** Empty input? Duplicate requests? Expected scale? Error versus partial result?
3. **Write a tiny example in a comment.** It prevents solving the wrong problem.
4. **Name the function and types.** A good signature makes the code smaller.
5. **Build happy path first.** No premature classes, queues, or frameworks.
6. **Test two normal cases and two ugly cases.**
7. **State the next production concern.** Timeout, idempotency, index, cache, or observability—only after correct code exists.

If you freeze: write the signature and one test first. Code follows the test.

---

## Your Small Python Starter Kit

Memorise these, not an entire framework.

### 1. Typed pure function

```python
from collections.abc import Iterable

def total_usage(events: Iterable[dict[str, object]]) -> dict[str, int]:
    totals: dict[str, int] = {}
    for event in events:
        org_id = event.get("org_id")
        tokens = event.get("tokens", 0)
        if not isinstance(org_id, str) or not isinstance(tokens, int) or tokens < 0:
            raise ValueError("event must contain a string org_id and non-negative integer tokens")
        totals[org_id] = totals.get(org_id, 0) + tokens
    return totals
```

Say: "I keep business logic pure so it is easy to test. The HTTP/database layer can call this function."

### 2. Minimal FastAPI endpoint

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, Field

app = FastAPI()

class CreateItem(BaseModel):
    name: str = Field(min_length=1, max_length=100)

@app.post("/items", status_code=201)
async def create_item(item: CreateItem) -> dict[str, str]:
    if item.name.lower() == "forbidden":
        raise HTTPException(status_code=400, detail="invalid name")
    return {"id": "generated-id", "name": item.name}
```

Say: "Pydantic validates untrusted input at the boundary. In a real service I would inject a repository/service instead of storing state in the route."

### 3. Async call with timeout and explicit failure

```python
import asyncio

async def fetch_profile(client, user_id: str) -> dict:
    try:
        response = await asyncio.wait_for(client.get(f"/profiles/{user_id}"), timeout=1.0)
        response.raise_for_status()
        return response.json()
    except TimeoutError as exc:
        raise RuntimeError("profile service timed out") from exc
```

Say: "I use async for I/O wait. If this call were CPU-heavy, I would not put it directly on the event loop."

### 4. Test first when unsure

```python
def test_total_usage_groups_events():
    events = [{"org_id": "a", "tokens": 3}, {"org_id": "a", "tokens": 2}]
    assert total_usage(events) == {"a": 5}

def test_total_usage_rejects_negative_tokens():
    import pytest
    with pytest.raises(ValueError):
        total_usage([{"org_id": "a", "tokens": -1}])
```

---

## 10 Micro-Implementations to Practise in an IDE

Do each in 20-30 minutes. Open a blank folder, write `solution.py` and `test_solution.py`, run it, then explain it aloud.

### 1. Usage Aggregator

**Prompt:** Given `{org_id, model, input_tokens, output_tokens}` events, return total tokens and request count per org.

**Must handle:** empty input, missing/invalid fields, repeated orgs.

**Follow-up:** return the top `k` organisations. Use `heapq.nlargest` or sort after correct aggregation; say complexity.

### 2. Sliding-Window Request Limiter (In-Memory)

**Prompt:** Allow at most `N` requests for each user in the last 60 seconds.

**Core:** `dict[user_id, deque[timestamp]]`; drop timestamps `<= now - window`, then allow iff deque length `< limit`.

**Say:** "This is correct in one process. For multiple API instances, the state must move to an atomic shared store such as Redis; I would not pretend this local dictionary scales."

### 3. TTL Cache Decorator / Class

**Prompt:** Cache a function result for 30 seconds.

**Core:** map key -> `(expiry, value)`; use `time.monotonic()` for elapsed time, not wall-clock time.

**Follow-up:** cap size or evict LRU; mention cache stampede if many callers miss together.

### 4. Retry With Backoff

**Prompt:** Retry a transient operation up to three times.

**Core:** retry only known transient exceptions; exponential delay plus jitter; re-raise the final exception.

**Do not:** retry validation errors or blindly retry non-idempotent payments/creates.

### 5. Async Fan-Out, Bounded

**Prompt:** Fetch profiles for 100 user IDs concurrently without overloading the downstream service.

**Core:** `asyncio.Semaphore(10)` around each request, `asyncio.gather`, return per-ID success/failure.

**Follow-up:** use a timeout and cancellation policy; do not make 100 unbounded requests just because `gather` exists.

### 6. Paginated API Function

**Prompt:** Return a page of projects after a cursor.

**Core:** deterministic order such as `(created_at, id)`; fetch `limit + 1`; next cursor comes from the last returned row.

**Say:** "I avoid offset pagination because inserts can cause duplicates/skips at scale."

### 7. SQL Analytics Query

**Prompt:** From `requests(org_id, created_at, cost)`, find the top 3 organisations by spend in the last 7 days.

```sql
SELECT org_id, SUM(cost) AS total_cost
FROM requests
WHERE created_at >= NOW() - INTERVAL '7 days'
GROUP BY org_id
ORDER BY total_cost DESC
LIMIT 3;
```

**Follow-up:** likely index `(created_at, org_id)` or choose based on real query plan/data distribution. Say `EXPLAIN ANALYZE` before claiming an index is right.

### 8. Parse and Validate JSON Lines

**Prompt:** Read a `.jsonl` log, skip malformed lines with an error counter, and write valid records grouped by org.

**Core:** stream line-by-line; never load an unbounded file all at once. Return both result and errors so failure is observable.

### 9. Fix a Blocking Async Bug

**Prompt:** An `async def` calls `requests.get()` or heavy CPU work and the server becomes slow. Fix it.

**Core:** use an async HTTP client for network I/O; put CPU-heavy work in a process/worker, not directly on the event loop.

### 10. Implement a Small C++ Class

**Prompt:** Make an `LRUCache` API with `get` and `put`, or a `TaskScheduler` that returns the next highest-priority task.

**Core:** write the class header first; choose `unordered_map + list` for LRU or `priority_queue` for scheduling; test update/empty/capacity cases.

Your DSA material already has the algorithm. The new practice is: create files, compile/run, write a tiny test, and explain decisions.

---

## A Mini Mock: How It Should Sound

**Interviewer:** "Implement a per-user limiter, then make it production-ready."

**You:**

> "I’ll start with an in-memory sliding-window version so the behaviour is clear. Input is user ID and current time; output is allow/deny. For each user I will retain only timestamps from the last minute. That makes each request amortized O(1), although memory is proportional to requests in the active window."

Write the small deque solution. Then say:

> "For multiple API instances, I would put the state in Redis and make prune/check/add atomic—typically with a Lua script or a Redis primitive chosen for the algorithm. I would key by authenticated user or API key rather than only IP, add a TTL to clean idle keys, and test concurrent requests at the limit boundary."

This is exactly the bridge from LeetCode thinking to engineering thinking.

---

## IDE Habits That Prevent Freezing

- Create the smallest runnable file immediately. `python solution.py` or a tiny `pytest` test is progress.
- Use autocomplete/docs freely unless the interviewer says no. Real engineers do.
- Narrate intent, not every keystroke: "I am separating validation from aggregation."
- If syntax escapes you, write pseudocode in a comment, then look up the exact standard-library name if allowed.
- Prefer a working simple solution; then name how you would make it distributed, durable, or faster.
- Keep a `scratchpad.md` in your practice folder with the commands you forget: virtualenv, pytest, `g++`, Docker logs.

## Four-Week Confidence Plan (20-30 min/day)

| Week | Focus |
|---|---|
| 1 | Tasks 1, 7, 8; get comfortable running a Python file and pytest |
| 2 | Tasks 2, 3, 4; practise time, state, and error cases |
| 3 | Tasks 5, 6, 9; practise async/API reasoning aloud |
| 4 | Task 10 + any two repeats under a 25-minute timer |

The win is not memorising this page. It is doing 8-10 tiny implementations in a real editor. After that, an IDE becomes familiar territory instead of a test of your worth.
