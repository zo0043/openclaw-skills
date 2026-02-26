# Architecture Teardowns

## Table of Contents
1. [Redis](#redis)
2. [Kafka](#kafka)
3. [SQLite](#sqlite)
4. [Git](#git)
5. [Nginx](#nginx)
6. [Kubernetes](#kubernetes)

---

## Redis

### Problem
Need a data structure server that is extremely fast for read/write operations, usable as cache, message broker, and lightweight database.

### Core Design Decisions

**1. Single-threaded event loop**
- Why: Eliminates lock contention, context switching overhead. Memory operations are already microsecond-level; threading overhead would dominate.
- Trade-off: Can't use multiple CPU cores for data operations (solved with multiple instances + cluster mode). Simpler code, easier to reason about.

**2. In-memory with optional persistence**
- Why: Memory is orders of magnitude faster than disk. Most use cases value speed over durability.
- AOF (Append-Only File): Every write logged, replay on restart. Trade-off: disk I/O per write vs full durability.
- RDB (Snapshot): Point-in-time snapshot via fork(). Trade-off: potential data loss between snapshots vs low overhead.
- Hybrid: RDB for base + AOF for recent changes. Best of both.

**3. Purpose-built data structures**
- Why: Generic KV stores treat everything as bytes. Redis understands lists, sets, sorted sets, hashes — enabling atomic operations on complex data without read-modify-write cycles.
- Trade-off: More complex implementation vs dramatically simpler client code.

**4. Cluster mode with hash slots**
- 16,384 hash slots distributed across nodes. Key → CRC16 → slot → node.
- Why 16,384: Small enough for efficient gossip protocol, large enough for resharding granularity.
- Trade-off: No cross-slot transactions vs horizontal scalability.

**5. Pub/Sub as first-class citizen**
- Fire-and-forget messaging built into the data store. No persistence for pub/sub messages.
- Trade-off: Simplicity and speed vs delivery guarantees (use Streams for persistent messaging).

### What Breaks at Scale
- Memory is the ceiling. Large datasets require cluster mode with its complexity.
- Pub/Sub doesn't scale well: each subscriber on each node receives all messages.
- Large keys (e.g., sorted set with millions of members) block the event loop during operations.
- Replication lag during heavy write loads causes stale reads on replicas.

### Key Lessons
- **Simplicity as a feature**: Single-threaded isn't a limitation, it's a design choice that eliminates entire classes of bugs.
- **Right tool for the right job**: Built-in data structures eliminate the impedance mismatch between application logic and storage.
- **Persistence is a spectrum**: Not everything needs ACID; choosing the right durability level unlocks massive performance gains.

---

## Kafka

### Problem
Need a distributed event streaming platform that handles high-throughput, durable message delivery with replay capability.

### Core Design Decisions

**1. Append-only log as the core abstraction**
- Why: Sequential writes are orders of magnitude faster than random writes on both HDD and SSD. Simplifies replication and retention.
- Trade-off: Can't delete individual messages efficiently (solved with retention policies and compaction).

**2. Consumer-driven offset management**
- Kafka doesn't track which messages each consumer has read. Consumers maintain their own offset.
- Why: Eliminates per-message state on the broker. Enables replay by resetting offset.
- Trade-off: Consumer must handle offset commits correctly (at-least-once by default).

**3. Partitions for parallelism**
- Topics split into partitions. Each partition is an ordered, append-only log on one broker.
- Ordering guaranteed within partition, not across partitions.
- Why: Parallelism without coordination. N partitions → N concurrent consumers.
- Trade-off: Cross-partition ordering requires application-level logic.

**4. Replication with ISR (In-Sync Replicas)**
- Each partition has a leader and followers. Only ISR members can become leader.
- acks=all: Wait for all ISR members. acks=1: Wait for leader only.
- Trade-off: Durability vs latency.

**5. Zero-copy transfer**
- Uses sendfile() to transfer data directly from disk to network socket, bypassing user space.
- Why: Reduces CPU usage and memory copies. Critical for high-throughput.
- Trade-off: Only works for unprocessed data transfer (no filtering at broker).

### What Breaks at Scale
- Partition rebalancing causes consumer pauses (stop-the-world during rebalance).
- Too many partitions increase leader election time and memory overhead on controller.
- Consumer lag monitoring becomes critical — slow consumers cause retention pressure.
- Cross-datacenter replication adds complexity (MirrorMaker 2).

### Key Lessons
- **Simplicity enables performance**: Append-only log is the simplest possible storage, yet enables the highest throughput.
- **Push complexity to the edges**: Broker stays simple; consumers handle their own state.
- **Sequential I/O is king**: Designing around sequential access patterns unlocks hardware capabilities.

---

## SQLite

### Problem
Need a full-featured relational database that runs embedded in the application process — no server, no configuration.

### Core Design Decisions

**1. Serverless (embedded) architecture**
- The database runs in the same process as the application. No IPC, no network.
- Why: Zero configuration, zero administration, zero latency for local access.
- Trade-off: No concurrent writes from multiple processes (single writer lock).

**2. Single file storage**
- Entire database in one cross-platform file. Easy to copy, backup, transfer.
- Why: Maximum portability. A database is just a file.
- Trade-off: Cannot leverage multiple disks, file size limited by OS.

**3. B-tree for both tables and indexes**
- Tables stored as B-trees keyed by rowid. Indexes are separate B-trees.
- Why: Balanced read/write performance, efficient range scans.
- Trade-off: Write amplification for large rows (entire page rewritten).

**4. WAL (Write-Ahead Logging) mode**
- Writes go to a separate WAL file; readers see the original database file.
- Why: Enables concurrent readers during writes. Readers never block writers.
- Trade-off: Slightly slower writes (write WAL + checkpoint), WAL file growth needs management.

**5. Extensive testing (100% branch coverage)**
- 150,000+ test cases, ~700x more test code than source code.
- Why: Used in safety-critical systems (aircraft, medical devices). Reliability is non-negotiable.
- Trade-off: Development velocity vs correctness guarantee.

### What Breaks at Scale
- Single writer: only one write transaction at a time.
- No built-in replication: application must handle distribution.
- Large databases (>1TB) hit practical limits.
- High-concurrency write workloads need a client-server database.

### Key Lessons
- **Know your constraints**: SQLite embraces the single-machine constraint and optimizes everything for it.
- **Testing as architecture**: The extreme test coverage IS the reliability guarantee, not the code itself.
- **Good enough is often perfect**: For most applications, SQLite is more than sufficient. Reaching for PostgreSQL "just in case" adds unnecessary complexity.

---

## Git

### Problem
Need a distributed version control system that handles branching, merging, and collaboration without a central server requirement.

### Core Design Decisions

**1. Content-addressable storage (SHA-1 hash)**
- Every object (blob, tree, commit) is identified by the SHA-1 hash of its content.
- Why: Automatic deduplication, integrity verification, and immutability.
- Trade-off: Hash collisions (extremely rare), SHA-1 weakness (migrating to SHA-256).

**2. Snapshots, not diffs**
- Each commit stores a complete snapshot of the tree (via content-addressed pointers).
- Why: Fast checkout of any version without replaying diffs. Branching is just creating a new pointer.
- Trade-off: Storage efficiency (solved by packfiles with delta compression).

**3. DAG (Directed Acyclic Graph) for history**
- Commits form a DAG. Each commit points to parent(s).
- Why: Enables non-linear history (branches, merges) naturally. No need for centralized branch tracking.
- Trade-off: History can be complex to understand (rebase vs merge debates).

**4. Distributed-first with no central authority**
- Every clone is a full repository with complete history.
- Why: Works offline, fast local operations, no single point of failure.
- Trade-off: Large repositories consume significant local storage.

**5. Three-stage architecture (working directory → staging area → repository)**
- The staging area (index) allows partial commits and careful change curation.
- Why: Fine-grained control over what gets committed.
- Trade-off: Added complexity compared to simpler VCS (confusing for beginners).

### Key Lessons
- **Content-addressing is powerful**: Once you hash content, deduplication, integrity, and caching come for free.
- **Make the common case fast**: Branch creation is O(1) — just write 41 bytes. This changes how people use branches.
- **Distributed design enables centralized workflows**: GitHub is centralized, but built on a distributed foundation. Best of both worlds.

---

## Nginx

### Problem
Need a high-performance HTTP server and reverse proxy that handles massive concurrent connections.

### Core Design Decisions

**1. Event-driven, non-blocking I/O**
- Single master process + N worker processes, each handling thousands of connections via epoll/kqueue.
- Why: One thread per connection (Apache model) wastes memory and CPU at scale. Event loop handles 10K+ connections per worker.
- Trade-off: Can't use blocking operations in request processing (must be async).

**2. Master-worker process model**
- Master: manages configuration, spawns workers, handles signals.
- Workers: handle actual requests, independent of each other.
- Why: Worker crash doesn't affect other workers. Graceful reload: spawn new workers with new config, drain old ones.
- Trade-off: IPC between workers is limited (shared memory for caches).

**3. Configuration-driven behavior**
- Declarative configuration instead of code.
- Why: Most use cases (routing, proxying, caching, SSL) are configuration, not logic.
- Trade-off: Complex logic requires Lua modules or external services.

### Key Lessons
- **Event-driven beats thread-per-connection**: At high concurrency, the event loop model wins decisively.
- **Process isolation for reliability**: Crashing one worker doesn't take down the server.
- **Graceful reloads enable zero-downtime**: The ability to reload configuration without dropping connections is operationally transformative.

---

## Kubernetes

### Problem
Need a platform to orchestrate containerized applications across a cluster with automated deployment, scaling, and management.

### Core Design Decisions

**1. Declarative desired state + reconciliation loop**
- Users declare WHAT they want (3 replicas of this container). Controllers reconcile current state → desired state.
- Why: Self-healing. If a pod dies, controller notices and creates a new one. No manual intervention.
- Trade-off: Eventually consistent — there's always a gap between desired and actual state.

**2. Everything is an API object**
- Pods, Services, Deployments, ConfigMaps — all CRUD resources with a uniform API.
- Why: Extensible via Custom Resource Definitions (CRDs). Ecosystem can add new resource types.
- Trade-off: Indirection layers add complexity (Deployment → ReplicaSet → Pod).

**3. etcd as the single source of truth**
- All cluster state stored in etcd (distributed KV store with strong consistency via Raft).
- Why: Consistent view of cluster state, watch API for change notifications.
- Trade-off: etcd is the critical path. If etcd is slow or down, the entire control plane suffers.

**4. Flat networking with CNI plugins**
- Every pod gets a unique IP. Pods can communicate directly without NAT.
- Why: Simplifies service discovery and communication. Applications don't need to know about containers.
- Trade-off: Network plugin complexity (Calico, Cilium, Flannel each with different trade-offs).

**5. Labels and selectors for loose coupling**
- Services find pods via label selectors, not direct references.
- Why: Pods are ephemeral. Labels provide stable grouping over changing pod instances.
- Trade-off: Debugging can be hard (which pods match this selector?).

### Key Lessons
- **Declarative > imperative for infrastructure**: Reconciliation loops make systems self-healing.
- **Uniform APIs enable ecosystems**: CRDs turned Kubernetes into a platform for building platforms.
- **Complexity is real**: Kubernetes solves hard problems but introduces its own. Right-size your orchestration needs.
