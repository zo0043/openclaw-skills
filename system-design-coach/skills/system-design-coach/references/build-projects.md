# Build Projects

## Table of Contents
1. [Beginner Projects](#beginner-projects)
2. [Intermediate Projects](#intermediate-projects)
3. [Advanced Projects](#advanced-projects)

---

## Beginner Projects

### B1: Key-Value Store
Build a persistent KV store from scratch.

**Learning goals**: Storage engines, data structures, durability

**Requirements**:
- GET/SET/DELETE operations
- Persistence to disk (survives restart)
- Support for concurrent reads

**Phases**:
1. In-memory hash map with simple file persistence
2. Add WAL (Write-Ahead Log) for crash recovery
3. Implement LSM-tree or B-tree storage engine
4. Add bloom filters for read optimization
5. Benchmark: measure ops/sec, latency percentiles

**Chaos challenges**:
- Kill the process mid-write. Does it recover correctly?
- What happens with 10M keys? 100M?
- What if values are 1MB each?

**Recommended language**: Go or Rust (for learning systems programming) or Python (for faster iteration)

---

### B2: HTTP Load Balancer
Build a reverse proxy that distributes traffic across backends.

**Learning goals**: Networking, concurrency, health checks

**Requirements**:
- Round-robin and least-connections algorithms
- Health check with automatic backend removal/addition
- Connection pooling

**Phases**:
1. Basic TCP proxy forwarding to one backend
2. Add multiple backends with round-robin
3. Implement health checks (HTTP 200 check every 5s)
4. Add least-connections algorithm
5. Add weighted routing

**Chaos challenges**:
- Kill a backend mid-request. What happens?
- Slow backend (add 10s delay). Does it affect other backends?
- All backends down. How does it respond?

---

### B3: Message Queue
Build a simple persistent message queue.

**Learning goals**: Producer-consumer pattern, durability, delivery guarantees

**Requirements**:
- Publish messages to named queues
- Subscribe and consume messages
- At-least-once delivery
- Persistence (messages survive restart)

**Phases**:
1. In-memory queue with TCP interface
2. Add file-based persistence
3. Implement consumer acknowledgment
4. Add dead-letter queue for failed messages
5. Support multiple consumers (fan-out and competing consumers)

**Chaos challenges**:
- Consumer crashes without acknowledging. Is message redelivered?
- Producer sends duplicate. How to handle?
- Queue has 1M pending messages. Memory usage?

---

## Intermediate Projects

### I1: Distributed Cache
Build a distributed cache with consistent hashing.

**Learning goals**: Consistent hashing, cluster membership, cache invalidation

**Requirements**:
- GET/SET/DELETE with TTL
- Multiple nodes with consistent hash ring
- Automatic rebalancing when nodes join/leave
- Cache invalidation

**Phases**:
1. Single-node cache with LRU eviction
2. Add consistent hashing for multi-node distribution
3. Implement virtual nodes for better balance
4. Add gossip protocol for membership management
5. Client-side library with connection pooling

**Chaos challenges**:
- Kill a node. How fast does the ring rebalance?
- Add a node under load. How many keys need to move?
- Network partition between two groups of nodes.

---

### I2: Search Engine
Build a full-text search engine.

**Learning goals**: Inverted index, text processing, ranking

**Requirements**:
- Index documents (title + body)
- Full-text search with TF-IDF ranking
- Support boolean queries (AND, OR, NOT)

**Phases**:
1. Build inverted index from documents
2. Implement TF-IDF scoring
3. Add tokenizer with stemming and stop words
4. Support phrase queries
5. Add incremental index updates (add/remove documents without rebuilding)

**Chaos challenges**:
- Index 1M documents. Query latency?
- Document added — how fast until it's searchable?
- Index corrupts. Can you rebuild from source documents?

---

### I3: Task Scheduler
Build a distributed task scheduler with dependencies.

**Learning goals**: DAG execution, distributed coordination, failure handling

**Requirements**:
- Schedule one-shot and recurring tasks
- Task dependencies (task B runs after task A completes)
- Retry on failure with configurable backoff
- Task status and history

**Phases**:
1. Single-worker scheduler with cron-like scheduling
2. Add task dependencies (DAG)
3. Multi-worker with task distribution
4. Implement retry with exponential backoff
5. Add monitoring dashboard (task status, success rate, latency)

**Chaos challenges**:
- Worker crashes mid-task. Is it retried? Not double-executed?
- Circular dependency in DAG. Detected?
- 10K tasks scheduled for the same second. What happens?

---

## Advanced Projects

### A1: Mini Database
Build a simplified relational database.

**Learning goals**: Query parsing, execution plans, transactions, concurrency control

**Requirements**:
- CREATE TABLE, INSERT, SELECT with WHERE
- B-tree index for primary key
- Basic transaction support (BEGIN, COMMIT, ROLLBACK)
- MVCC for concurrent reads

**Phases**:
1. Table storage with row-based pages
2. SQL parser for basic queries
3. B-tree index for primary key lookups
4. Add WHERE clause filtering
5. Implement MVCC for snapshot isolation
6. Add WAL for crash recovery

**Chaos challenges**:
- Two transactions write the same row. Correct behavior?
- Crash between WAL write and data write. Recovery?
- Table with 10M rows, no index. vs with index. Benchmark.

---

### A2: Raft Consensus
Implement the Raft consensus algorithm.

**Learning goals**: Distributed consensus, leader election, log replication

**Requirements**:
- Leader election with term-based voting
- Log replication from leader to followers
- Safety guarantee: committed entries are never lost

**Phases**:
1. Implement leader election (RequestVote RPC)
2. Implement log replication (AppendEntries RPC)
3. Implement commit and apply
4. Add log compaction (snapshotting)
5. Handle membership changes

**Chaos challenges**:
- Kill the leader. How fast is re-election?
- Network partition: {A,B} | {C,D,E}. Both sides' behavior?
- Slow follower (100ms latency). Impact on commit?

**Reference**: The Raft paper (In Search of an Understandable Consensus Algorithm) is one of the best-written systems papers. Read it first.

---

### A3: AI Inference Gateway
Build a gateway that routes requests across multiple LLM providers.

**Learning goals**: AI-era infrastructure, routing, cost optimization, streaming

**Requirements**:
- Proxy requests to multiple LLM APIs (OpenAI, Anthropic, local models)
- Intelligent routing based on cost, latency, and model capability
- Streaming response support (SSE)
- Request caching for identical prompts
- Rate limiting per user/tenant

**Phases**:
1. Simple proxy to one provider with streaming
2. Add multiple providers with round-robin routing
3. Implement semantic caching (cache similar prompts)
4. Add cost-aware routing (use cheap model when possible)
5. Add fallback chains (if primary fails, try secondary)
6. Usage tracking and budget enforcement

**Chaos challenges**:
- Provider returns 500. Fallback behavior?
- Streaming response interrupted mid-stream. Client handling?
- Budget exceeded mid-request. Graceful handling?

**Why this project matters**: This is the actual infrastructure being built at AI companies right now. Directly applicable to real jobs.
