# Chapter 2: Non-Functional Requirements

## TL;DR

Dedicated chapter on NFRs: security, scalability, reliability, maintainability, and performance. Expands on concepts introduced in Chapter 1 with a structured taxonomy.

## Performance

### Latency vs Throughput

| Metric | Definition | Unit examples |
|--------|-----------|---------------|
| **Latency** | Duration of a single operation | ms, seconds |
| **Throughput** | Rate of work completed | req/s, records/s, bytes/s |

These aren't independent — they trade off against each other. Push throughput high enough and latency degrades due to queueing, contention, and resource saturation.

- **Batching** improves throughput (amortize overhead) but hurts individual latency
- **Streaming** optimizes for latency at the cost of throughput

Example: Lambda has low per-invocation latency, but limited concurrency is a throughput ceiling. Hit it and requests queue — latency spikes.

## Reliability

### Faults vs Failures

A **fault** is one component deviating from spec. A **failure** is the system as a whole stopping serving users. Goal: **fault-tolerant** systems — faults will happen, failures shouldn't.

### Three Fault Categories

| Type | Example | Mitigation |
|------|---------|------------|
| **Hardware** | Disk dies, power loss, network cable | Redundancy (RAID, multi-AZ, failover) |
| **Software** | Bug triggered by specific input, memory leak, cascading failure | Testing, process isolation, crash-and-restart |
| **Human** | Bad config deploy, wrong command, missed step | Guardrails, rollback, sandbox environments, observability |

Hardware faults are random and independent. Software and human faults **correlate and cascade** — a bad deploy hits every node simultaneously. That's why they're harder.

### Tolerate vs Prevent

Most faults (node down, network blip) — tolerate through redundancy and graceful degradation. Some faults (data corruption, security breach) — **prevention is the only option**, because recovery may be impossible or the damage undetectable.

## Scalability

Scalability isn't binary — it's always "scalable *with respect to what*?" You describe load with parameters specific to your system (req/s, read/write ratio, fan-out factor), then ask what happens to performance as those parameters grow.

The Twitter home timeline is the chapter's standout example. The fan-out shape of the follow graph — not raw volume — drove the entire architecture. Normal users fan out on write (push tweets to followers' caches), celebrities fan out on read (fetched at timeline load), and the system merges both paths. One follower_count threshold determines the whole write path.

Deep dive: [Fan-Out Primer](../applied/fan-out-primer.md)
