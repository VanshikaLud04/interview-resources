# AI Coding Questions — By Company

## Anthropic
- **Inference Engine**
  - **Task Type**: Build
  - **Key Skills Tested**: Systems design, optimizing bottlenecks, algorithmic efficiency.
  - **Approach Hints**: Focus on efficient token generation, memory management (KV caching), and batching requests.
- **Debug Extremely Randomized Trees**
  - **Task Type**: Repair
  - **Key Skills Tested**: Machine Learning knowledge (Random Forests), debugging logic errors.
  - **Approach Hints**: Understand node splitting criteria, variance vs bias, and tree building recursion.
- **Repair Agent Turn Reduction**
  - **Task Type**: Optimize/Repair
  - **Key Skills Tested**: AI agent architecture, tool usage optimization, state management.
  - **Approach Hints**: Look for redundant tool calls or infinite loops in agent reasoning loops.

## Amazon
- **Repair Wallet Loan Marketplace**
  - **Task Type**: Repair
  - **Key Skills Tested**: Business logic, edge cases, financial math.
  - **Approach Hints**: Trace the lifecycle of a loan, check interest calculations, and validate state transitions.
- **Repair MovieDB Recommendations**
  - **Task Type**: Repair
  - **Key Skills Tested**: Recommendation algorithms (collaborative filtering, content-based), data structures.
  - **Approach Hints**: Check for off-by-one errors, cold start problem handling, or incorrect similarity metrics.
- **Repair Recurring Wallet Payments**
  - **Task Type**: Repair
  - **Key Skills Tested**: Concurrency, scheduling, state machines.
  - **Approach Hints**: Ensure idempotent transactions, handle timezone issues, and check retry mechanisms.

## DoorDash
- **Build Refund DAG with Local HTTP Services**
  - **Task Type**: Build
  - **Key Skills Tested**: Distributed systems, Directed Acyclic Graphs, API design.
  - **Approach Hints**: Implement topological sort for execution order, handle HTTP failures gracefully with retries.

## OpenAI
- **Repair Django Rate Limiter**
  - **Task Type**: Repair
  - **Key Skills Tested**: Web framework internals, caching (Redis), concurrency.
  - **Approach Hints**: Identify race conditions in token bucket or sliding window implementations, check atomicity.

## Snap
- **Build Persistent Chat Backend**
  - **Task Type**: Build
  - **Key Skills Tested**: Database schema design, WebSockets/SSE, scalability.
  - **Approach Hints**: Design for high write throughput, efficient pagination for history, handling disconnected clients.

## Baseten
- **Durable Key-Value Store**
  - **Task Type**: Build
  - **Key Skills Tested**: File I/O, serialization, crash recovery (WAL).
  - **Approach Hints**: Use a write-ahead log for durability before updating in-memory state.
- **Parallelize API Calls Thread Pool**
  - **Task Type**: Optimize
  - **Key Skills Tested**: Multithreading, synchronization primitives.
  - **Approach Hints**: Use `ThreadPoolExecutor`, handle exceptions inside futures, optimize worker count based on I/O bound nature.

## Headway
- **Fixed-Window Rate Limiter**
  - **Task Type**: Build
  - **Key Skills Tested**: Time manipulation, atomic operations.
  - **Approach Hints**: Implement counters keyed by time windows, clear old windows to save memory.
- **Repair Insurance Eligibility Projection**
  - **Task Type**: Repair
  - **Key Skills Tested**: Complex business logic, edge case handling.
  - **Approach Hints**: Carefully read the rule engine logic, check boundary conditions for dates and coverage amounts.

## Scale AI
- **CSV-to-JSON LLM Categorization**
  - **Task Type**: Build
  - **Key Skills Tested**: Data parsing, API integration, prompt engineering.
  - **Approach Hints**: Batch requests to the LLM API, parse responses robustly, handle rate limits.

## Mercury
- **Build Python CRUD API**
  - **Task Type**: Build
  - **Key Skills Tested**: Web framework proficiency (FastAPI/Flask), database ORM, REST principles.
  - **Approach Hints**: Ensure proper HTTP status codes, input validation, and secure database queries (avoid SQLi).

## Coinbase
- **In-Memory Database**
  - **Task Type**: Build
  - **Key Skills Tested**: Data structures, transaction management (Begin/Commit/Rollback).
  - **Approach Hints**: Implement a stack of nested transactions, tracking changes to rollback efficiently.
