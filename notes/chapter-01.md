# Chapter 1: Reliable, Scalable, and Maintainable Applications

## TL;DR

Most applications are **data-intensive**, not compute-intensive. The chapter frames three fundamental concerns — reliability, scalability, maintainability — as competing tensions that shape every architectural decision. Introduces OLTP vs OLAP as distinct workload categories that demand different system designs.

## Key Concepts

### Three Pillars

| Pillar | Core question |
|--------|---------------|
| **Reliability** | Does the system work correctly when things go wrong? |
| **Scalability** | Can the system handle growth in data, traffic, or complexity? |
| **Maintainability** | Can the system be understood, extended, and operated over time? |

These aren't independent — optimizing one often costs another. Fast iteration (maintainability) can introduce bugs (reliability). Scaling out (scalability) adds operational complexity (maintainability).

### Reliability

- **Faults vs failures** — a fault is a component deviating from spec; a failure is the system stopping. Goal: fault-tolerant, not fault-free.
- Three fault categories: hardware faults, software errors, human errors.
- Hardware faults are well-understood (redundancy). Software and human faults are harder — they correlate and cascade.

### Scalability

- **Load parameters** — describe load with numbers specific to your system (requests/sec, read/write ratio, active users, fan-out). Twitter's home timeline is the canonical example: the fan-out shape of the workload drives the architecture, not just volume.
- **Percentiles over averages** — p50, p95, p99, p999. Mean latency hides tail pain. Tail latency amplification: in fan-out architectures, the slowest backend call sets the user-perceived latency.
- Scaling up (vertical) vs scaling out (horizontal) — no universal answer, depends on load shape.

### Maintainability

Three design principles:
- **Operability** — make it easy for ops to keep things running (monitoring, deploys, runbooks).
- **Simplicity** — manage complexity through good abstractions. Accidental complexity compounds silently.
- **Evolvability** — make change easy. Requirements will change; the system should accommodate that without rewrites.

### OLTP vs OLAP

| | OLTP | OLAP |
|---|---|---|
| **Pattern** | Many small reads/writes | Few large scans/aggregations |
| **Users** | End users via applications | Analysts, dashboards, reports |
| **Latency** | Low (ms) | Higher acceptable (seconds-minutes) |
| **Data shape** | Current state, row-oriented | Historical, column-oriented |

Key distinction: it's the **access pattern** that defines OLTP vs OLAP, not the audience. A user-facing dashboard aggregating millions of rows is OLAP workload even though an end user sees it.

## Applied Connections

- **Tail latency amplification** — CloudFront → Varnish → HAProxy → origin. Each hop is a fan-out point. One slow origin response dominates the user experience at p99.
- **OLTP** — DynamoDB serving React apps, API responses for user-facing traffic.
- **OLAP** — Datadog aggregating APM spans across millions of traces for dashboards.
- **Operability** — Datadog RUM/APM, GitLab CI pipelines, CDK-managed infra. All reduce operational burden.
- **Accidental complexity** — every abstraction layer (CDK, CloudFront behaviors, caching rules) can compound if not managed.

## Open Questions

- How does 2e handle the shift toward serverless (Lambda, edge compute) in the scalability discussion? Traditional vertical/horizontal framing doesn't map cleanly.
- Where does Kleppmann draw the line between "tolerate faults" and "prevent faults"? In practice, some faults (data corruption) need prevention, not tolerance.
- The OLTP/OLAP boundary is blurring — real-time analytics, HTAP databases. Does 2e address this convergence?
