# Design Principles

## Table of Contents
1. [Fundamentals](#fundamentals)
2. [Data](#data)
3. [Scalability](#scalability)
4. [Reliability](#reliability)
5. [Simplicity](#simplicity)
6. [Operations](#operations)
7. [AI-Era](#ai-era)

---

## Fundamentals

### Make It Work, Make It Right, Make It Fast (In That Order)
Premature optimization and over-engineering cause more damage than under-engineering. Ship v1 that solves the problem. Improve based on real data, not speculation.

### Defer Decisions You're Uncertain About
Use abstraction layers to isolate uncertain decisions. When you must decide, you'll have more information. "The best architectures allow major decisions to be deferred." — Robert C. Martin

### Understand the Problem Before Designing the Solution
80% of design failures come from solving the wrong problem. Spend disproportionate time on requirements clarification. Ask "why" five times.

### Design for the Expected Case, Handle the Edge Cases
Don't let edge cases drive your core architecture. Handle them, but don't let them dictate the main path.

## Data

### Data Model Is Destiny
Your data model determines your system's ceiling. Everything else can be refactored; data model changes are the most painful and risky migrations. Get it right early, or at least get it flexible.

### Writes Are Hard, Reads Are Easy
Most scaling challenges come from writes. Design your write path first. Reads can usually be solved with caching, replication, and denormalization.

### Don't Lose Data. Everything Else Is Negotiable.
Data loss is often irreversible. Latency spikes recover. Downtime ends. Data loss is forever. Design durability as a hard constraint, performance as an optimization.

### Schema-on-Write vs Schema-on-Read
Choose explicitly. Schema-on-write (SQL) catches errors early. Schema-on-read (NoSQL) gives flexibility. Most systems need both at different layers.

## Scalability

### Scale the Bottleneck, Not Everything
Identify the actual bottleneck before scaling. Scaling non-bottleneck components wastes money and adds complexity. Measure first, always.

### Stateless Services Scale Horizontally
Keep state out of your application servers. Push it to databases, caches, or message queues. Stateless services can be scaled by adding instances.

### Partition for Writes, Replicate for Reads
Sharding (partitioning) increases write capacity. Replication increases read capacity and availability. Most systems need both.

### Cache Aggressively, Invalidate Carefully
Caching is the easiest performance win. Cache invalidation is one of the hardest problems in CS. Choose your invalidation strategy deliberately: TTL, event-driven, or version-based.

## Reliability

### Failure Is Not an Exception, It's the Norm
Design assuming everything will fail: servers, networks, disks, entire data centers. The question isn't "will it fail?" but "what happens when it fails?"

### Timeouts on Everything
Every network call needs a timeout. Every database query needs a timeout. No timeout = potential infinite hang = cascading failure.

### Retry with Backoff and Jitter
Retry failures, but with exponential backoff and random jitter to avoid thundering herd. Set a maximum retry count. Make retried operations idempotent.

### Circuit Breakers Prevent Cascading Failures
When a downstream service is failing, stop calling it. Give it time to recover. Better to return a degraded response than to take down the entire system.

### Graceful Degradation Over Hard Failure
When part of the system fails, degrade functionality rather than failing completely. Show cached data, disable non-essential features, return partial results.

## Simplicity

### Complexity Is the Enemy
A system's complexity ceiling = the team's cognitive capacity. The smartest architecture that the team can't understand is worse than a simple one they can operate confidently.

### Fewer Moving Parts
Every component is a potential failure point. Every integration is a coupling point. Every service is an operational burden. Add components only when the benefit clearly outweighs the cost.

### Boring Technology Is a Feature
Use proven, well-understood technology for core infrastructure. Save your "innovation tokens" for the parts that differentiate your product. PostgreSQL > latest NewDB for almost everything.

### Avoid Distributed Systems Until You Must
A well-tuned single server handles more than most people think. Distributed systems introduce partial failure, network partitions, and consistency challenges. Don't distribute prematurely.

## Operations

### Observability Is Not Optional
If you can't measure it, you can't fix it, and you definitely can't improve it. Instrument from day one: metrics, logs, traces.

### Quantify, Don't Guess
"I think it's fast enough" is not acceptable. "p99 latency is 47ms under 1000 QPS" is. Make decisions based on data, not intuition.

### Design for Operability
The team that builds it runs it. Design with operational tasks in mind: deployment, rollback, scaling, debugging, data migration.

### Automate Toil
If you do it more than twice, automate it. Manual operations are error-prone and don't scale. Automation is a form of documentation.

## AI-Era

### AI Is Infrastructure, Not Magic
Treat LLMs like any other service: with SLAs, fallbacks, cost budgets, and monitoring. Don't architect around the assumption that AI always works correctly.

### Design for Model Swappability
Models improve rapidly. Today's best model is next quarter's baseline. Abstract the model layer so you can swap without redesigning.

### Latency Budgets for AI Pipelines
AI calls are slow compared to traditional APIs. Budget latency explicitly. Use streaming, caching, and async patterns to keep UX responsive.

### Human-in-the-Loop for High-Stakes Decisions
AI should augment, not replace, human judgment for decisions with significant consequences. Design review/approval points into your AI workflows.

### Cost Is a First-Class Constraint
AI inference is expensive. Design cost controls from the start: caching identical queries, routing to cheaper models when possible, setting per-user/per-tenant budgets.
