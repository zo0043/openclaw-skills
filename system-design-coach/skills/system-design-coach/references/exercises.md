# Design Exercises

## Table of Contents
1. [Junior Level](#junior-level)
2. [Mid Level](#mid-level)
3. [Senior Level](#senior-level)
4. [Staff Level](#staff-level)

---

## Junior Level

### J1: URL Shortener
Design a URL shortening service (like bit.ly).
- **Core requirements**: Shorten URLs, redirect to original, analytics
- **Estimated scale**: 100M URLs, 10K new/day, 100K redirects/day
- **Key areas to probe**: ID generation, storage choice, caching, redirect performance
- **Common mistakes**: Not considering hash collisions, missing analytics pipeline, ignoring expiration

### J2: Paste Service
Design a text/code paste service (like Pastebin).
- **Core requirements**: Create pastes, read pastes, expiration, syntax highlighting
- **Estimated scale**: 5M pastes, 1K new/day
- **Key areas to probe**: Storage (blob vs DB), content limits, rate limiting
- **Common mistakes**: Storing large text in relational DB, no size limits

### J3: Rate Limiter
Design a distributed rate limiting service.
- **Core requirements**: Per-user rate limiting, multiple strategies, low latency
- **Estimated scale**: 1M users, 10K RPS
- **Key areas to probe**: Algorithm choice, distributed state, clock synchronization
- **Common mistakes**: Race conditions in counters, ignoring distributed consistency

## Mid Level

### M1: News Feed
Design a social media news feed (like Twitter/Weibo).
- **Core requirements**: Post, follow, timeline generation, ranking
- **Estimated scale**: 100M users, 10K posts/sec, 500K feed reads/sec
- **Key areas to probe**: Fan-out on write vs fan-out on read, ranking algorithm, caching strategy
- **Follow-up pressures**: Celebrity problem (millions of followers), real-time updates, content moderation

### M2: Chat System
Design a real-time messaging system (like WeChat/Slack).
- **Core requirements**: 1:1 chat, group chat, online status, message history
- **Estimated scale**: 50M DAU, 10K messages/sec
- **Key areas to probe**: Connection management (WebSocket), message ordering, delivery guarantees, group fan-out
- **Follow-up pressures**: Message search, file sharing, end-to-end encryption

### M3: Notification System
Design a multi-channel notification system (push, SMS, email).
- **Core requirements**: Multi-channel delivery, templates, preferences, rate limiting
- **Estimated scale**: 10M notifications/day
- **Key areas to probe**: Priority queues, deduplication, retry strategies, provider failover
- **Follow-up pressures**: User preferences, A/B testing, analytics

### M4: Search Autocomplete
Design a search autocomplete/suggestion system.
- **Core requirements**: Real-time suggestions as user types, personalization, trending
- **Estimated scale**: 100M users, 10K QPS
- **Key areas to probe**: Trie data structure, ranking signals, update frequency, caching
- **Follow-up pressures**: Multi-language, personalization, abuse prevention

## Senior Level

### S1: Distributed Task Scheduler
Design a reliable distributed task scheduling system (like Airflow/Temporal).
- **Core requirements**: Schedule tasks (cron + one-shot), retry, dependencies, monitoring
- **Estimated scale**: 10M tasks/day, 1K concurrent workers
- **Key areas to probe**: Exactly-once execution, task deduplication, failure handling, worker management
- **Follow-up pressures**: DAG dependencies, dynamic scaling, multi-region

### S2: Object Storage
Design an S3-like object storage system.
- **Core requirements**: PUT/GET/DELETE objects, buckets, metadata, versioning
- **Estimated scale**: 100PB data, 100K RPS
- **Key areas to probe**: Data placement, replication, consistency, garbage collection, erasure coding
- **Follow-up pressures**: Cross-region replication, lifecycle policies, access control

### S3: Real-time Analytics Platform
Design a real-time event analytics system (like Mixpanel/Amplitude).
- **Core requirements**: Event ingestion, real-time aggregation, querying, dashboards
- **Estimated scale**: 1M events/sec, 7-day retention for real-time, 1-year for batch
- **Key areas to probe**: Lambda/Kappa architecture, time-series storage, pre-aggregation, query engine
- **Follow-up pressures**: Funnel analysis, cohort analysis, arbitrary dimensional queries

### S4: Payment System
Design a payment processing system.
- **Core requirements**: Process payments, refunds, reconciliation, ledger
- **Estimated scale**: 1M transactions/day
- **Key areas to probe**: Idempotency (critical!), double-entry ledger, provider failover, PCI compliance
- **Follow-up pressures**: Multi-currency, partial refunds, fraud detection, regulatory compliance

## Staff Level

### ST1: Multi-Region Database
Design a globally distributed database with tunable consistency.
- **Core requirements**: Multi-region writes, tunable consistency, automatic failover
- **Estimated scale**: Global, sub-100ms reads everywhere, cross-region writes
- **Key areas to probe**: Consensus protocol, conflict resolution, clock synchronization (TrueTime vs HLC), data placement
- **Follow-up pressures**: Regulatory data residency, cost optimization, schema evolution

### ST2: AI Inference Platform
Design a multi-model AI inference serving platform.
- **Core requirements**: Serve multiple LLMs, auto-scaling, A/B testing, cost optimization
- **Estimated scale**: 10K RPS, multiple model sizes (7B to 400B+)
- **Key areas to probe**: GPU scheduling, model loading/unloading, batching, routing (cost vs quality), caching
- **Follow-up pressures**: Streaming responses, tool calling, RAG integration, multi-tenant isolation, budget controls

### ST3: Global Content Delivery Platform
Design a CDN with edge compute capabilities.
- **Core requirements**: Static + dynamic content delivery, edge functions, cache management
- **Estimated scale**: 10M RPS globally, 200+ edge locations
- **Key areas to probe**: Cache hierarchy, cache invalidation, edge compute isolation, traffic routing, origin shield
- **Follow-up pressures**: Video streaming (HLS/DASH), DDoS protection, real-time purge, A/B testing at edge

### ST4: Event-Driven Microservices Platform
Design the platform infrastructure for an event-driven microservices ecosystem.
- **Core requirements**: Service discovery, event bus, schema registry, observability, deployment
- **Estimated scale**: 200 services, 50 teams, 1M events/sec
- **Key areas to probe**: Event schema evolution, exactly-once processing, distributed tracing, team autonomy vs platform consistency
- **Follow-up pressures**: Multi-tenancy, cost attribution, compliance audit trail, gradual migration from monolith

---

## Evaluation Rubric

Score each dimension 1-5:

| Dimension | 1 (Weak) | 3 (Adequate) | 5 (Strong) |
|-----------|----------|---------------|-------------|
| Requirements | Jumps to solution | Asks basic questions | Clarifies ambiguity, identifies non-obvious requirements |
| High-Level | Missing components | Reasonable architecture | Clean separation, clear data flow |
| Deep-Dive | Surface-level only | Explains one component well | Deep on the hardest part, with alternatives |
| Trade-offs | No awareness | Mentions trade-offs | Quantifies trade-offs, justifies choices with data |
| Scalability | Ignores scale | Handles 10x | Handles 100x with graceful degradation plan |
| Communication | Disorganized | Logical flow | Structured, concise, drives the conversation |
