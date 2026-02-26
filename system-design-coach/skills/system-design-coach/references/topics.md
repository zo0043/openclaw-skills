# Topics Reference

## Table of Contents
1. [Fundamentals](#fundamentals)
2. [Data & Storage](#data--storage)
3. [Distributed Systems](#distributed-systems)
4. [Scalability Patterns](#scalability-patterns)
5. [API & Communication](#api--communication)
6. [Reliability & Operations](#reliability--operations)
7. [AI-Era Patterns](#ai-era-patterns)

---

## Fundamentals

### Client-Server Architecture
The basic building block. Client sends requests, server processes and responds. Understand before everything else.
- Key concepts: stateless vs stateful servers, request-response cycle, connection management
- Trade-off: stateless (simpler scaling) vs stateful (better UX, WebSocket)

### Load Balancing
Distribute traffic across multiple servers. Critical for horizontal scaling.
- Algorithms: round-robin, least connections, consistent hashing, weighted
- L4 (transport) vs L7 (application) load balancing
- Health checks and failover
- Trade-off: simplicity vs intelligence in routing

### Caching
Store computed results closer to the consumer. The single most impactful performance optimization.
- Cache-aside, read-through, write-through, write-behind
- Cache invalidation strategies (TTL, event-driven, versioned)
- Multi-level: browser → CDN → API gateway → application → database
- Trade-off: consistency vs latency

### Proxies
Forward proxy (client-side) vs reverse proxy (server-side).
- Use cases: load balancing, SSL termination, rate limiting, caching
- Examples: Nginx, HAProxy, Envoy, Cloudflare

### Hashing & Consistent Hashing
Distribute data across nodes with minimal redistribution on topology changes.
- Why naive modulo hashing fails when nodes change
- Virtual nodes for better distribution
- Applications: distributed caches, database sharding, CDN routing

## Data & Storage

### SQL vs NoSQL
Not "which is better" but "which constraints matter."
- SQL: strong consistency, relational data, complex queries, ACID
- NoSQL families: document (MongoDB), key-value (Redis), wide-column (Cassandra), graph (Neo4j)
- Trade-off: flexibility vs consistency guarantees

### Database Indexing
B-tree, LSM-tree, hash index — how databases find data fast.
- B-tree: good for reads, in-place updates
- LSM-tree: good for writes, sequential I/O, needs compaction
- Composite indexes, covering indexes, partial indexes
- Trade-off: read speed vs write speed vs storage

### Database Replication
Copy data across machines for availability and read scaling.
- Single-leader, multi-leader, leaderless
- Synchronous vs asynchronous replication
- Replication lag and its consequences
- Trade-off: consistency vs availability vs latency

### Database Sharding (Partitioning)
Split data across machines for write scaling and storage.
- Range-based, hash-based, directory-based
- Hotspot problem and rebalancing
- Cross-shard queries and joins
- Trade-off: query flexibility vs scalability

### Data Modeling
How you model data determines your system's ceiling. Hardest thing to change later.
- Normalization vs denormalization
- Event sourcing and CQRS
- Schema evolution strategies
- Time-series data patterns

### Blob & Object Storage
For large unstructured data: images, videos, logs, backups.
- S3-style object stores
- CDN integration for serving
- Lifecycle policies and tiered storage

## Distributed Systems

### CAP Theorem
In a network partition, choose consistency or availability. You cannot have both.
- CP systems: prioritize consistency (e.g., ZooKeeper, etcd)
- AP systems: prioritize availability (e.g., Cassandra, DynamoDB)
- PACELC: extends CAP to include latency trade-off when no partition

### Consensus Algorithms
How distributed nodes agree on a value.
- Raft: understandable, leader-based (etcd, CockroachDB)
- Paxos: theoretical foundation, harder to implement
- Use cases: leader election, configuration management, distributed locks

### Distributed Transactions
Maintaining consistency across multiple services/databases.
- 2PC (two-phase commit): strong but blocking
- Saga pattern: compensating transactions, eventually consistent
- Outbox pattern: reliable event publishing
- Trade-off: consistency guarantees vs availability and performance

### Event-Driven Architecture
Systems communicate through events rather than direct calls.
- Event sourcing: store events as source of truth
- Message queues (RabbitMQ) vs event streams (Kafka)
- At-least-once, at-most-once, exactly-once semantics
- Trade-off: decoupling and scalability vs complexity and debugging difficulty

### MapReduce & Batch Processing
Process large datasets in parallel across a cluster.
- Map: transform each record independently
- Reduce: aggregate results
- Modern alternatives: Spark, Flink, Dataflow
- When to use batch vs stream processing

### Stream Processing
Process data in real-time as it arrives.
- Windowing: tumbling, sliding, session
- Watermarks and late data handling
- Exactly-once semantics in streams
- Tools: Kafka Streams, Flink, Spark Streaming

## Scalability Patterns

### Horizontal vs Vertical Scaling
Scale out (add machines) vs scale up (bigger machine).
- Vertical: simpler, has a ceiling, single point of failure
- Horizontal: complex (state management, consistency), virtually unlimited
- Most systems: scale up until you can't, then scale out

### Microservices vs Monolith
Not a binary choice — it's a spectrum.
- Start monolith, extract services when boundaries are clear
- Service boundaries should align with team boundaries
- The distributed monolith anti-pattern
- Trade-off: deployment independence vs operational complexity

### Rate Limiting & Throttling
Protect systems from overload, fair resource allocation.
- Token bucket, leaky bucket, sliding window
- Client-side vs server-side vs gateway-level
- Graceful degradation strategies

### Circuit Breaker
Stop cascading failures by failing fast.
- States: closed → open → half-open
- Configuration: failure threshold, recovery timeout
- Combine with retry, timeout, and fallback

### Backpressure
When a downstream system can't keep up, signal upstream to slow down.
- Reactive streams pattern
- Buffer vs drop vs signal strategies
- Critical for pipeline architectures

## API & Communication

### REST vs gRPC vs GraphQL
- REST: ubiquitous, HTTP-based, resource-oriented
- gRPC: binary protocol, strongly typed, bidirectional streaming, service-to-service
- GraphQL: client-driven queries, reduces over/under-fetching, adds server complexity
- Trade-off: simplicity vs efficiency vs flexibility

### WebSocket & Server-Sent Events
Real-time communication patterns.
- WebSocket: bidirectional, persistent connection
- SSE: server-to-client only, simpler, auto-reconnect
- When to use which

### API Gateway
Single entry point for all client requests.
- Routing, authentication, rate limiting, transformation
- BFF (Backend for Frontend) pattern
- Trade-off: centralized control vs single point of failure

### Idempotency
Making operations safe to retry without side effects.
- Idempotency keys for payment systems
- Natural idempotency (PUT, DELETE) vs designed idempotency (POST with key)
- Critical for distributed systems with unreliable networks

## Reliability & Operations

### Monitoring & Observability
The three pillars: metrics, logs, traces.
- RED method (Rate, Errors, Duration) for services
- USE method (Utilization, Saturation, Errors) for resources
- Distributed tracing for request flow
- Alerting: symptom-based vs cause-based

### Chaos Engineering
Deliberately inject failures to find weaknesses.
- Start small: kill a process, add latency
- Game days: planned failure exercises
- Netflix Chaos Monkey philosophy

### Deployment Strategies
How to ship safely.
- Blue-green, canary, rolling, feature flags
- Progressive delivery
- Rollback strategies
- Trade-off: safety vs speed vs infrastructure cost

### Disaster Recovery
Plan for the worst.
- RPO (Recovery Point Objective): how much data loss is acceptable
- RTO (Recovery Time Objective): how fast to recover
- Hot standby, warm standby, cold standby
- Multi-region architecture

## AI-Era Patterns

### RAG (Retrieval-Augmented Generation)
Combine LLMs with external knowledge.
- Document chunking strategies
- Embedding models and vector similarity
- Hybrid search: vector + keyword
- Reranking pipelines
- Trade-off: context window usage vs retrieval accuracy

### Vector Databases
Specialized storage for embedding vectors.
- Approximate Nearest Neighbor (ANN) algorithms: HNSW, IVF
- Products: Pinecone, Milvus, Qdrant, pgvector
- When to use dedicated vector DB vs extension

### Model Serving & Inference
Deploy ML/LLM models for production use.
- Batching strategies for throughput
- Model quantization (INT8, INT4) for cost
- Speculative decoding, MoE routing
- Serving frameworks: vLLM, SGLang, TensorRT-LLM
- Trade-off: latency vs throughput vs cost vs quality

### Agent Architecture
LLMs that take actions in the real world.
- ReAct pattern: reason then act
- Tool calling and function routing
- Memory: short-term (context window) vs long-term (external storage)
- Multi-agent orchestration
- MCP (Model Context Protocol) for tool interop
- Trade-off: autonomy vs control vs cost

### Feature Store
Centralized store for ML features, bridging training and serving.
- Online store (low-latency serving) vs offline store (batch training)
- Feature freshness and consistency
- Products: Feast, Tecton

### AI Gateway / LLM Router
Manage multiple LLM providers and models.
- Load balancing across providers
- Fallback chains (primary → secondary model)
- Cost optimization routing
- Rate limiting and usage tracking
- Caching for identical prompts
