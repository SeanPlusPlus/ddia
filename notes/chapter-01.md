# Chapter 1: Reliable, Scalable, and Maintainable Applications

## TL;DR

Most applications are data-intensive, not compute-intensive. The chapter frames three fundamental concerns — reliability, scalability, maintainability — as competing tensions that shape every architectural decision. Also introduces OLTP vs OLAP as distinct workload categories demanding different system designs.

## Three Pillars

Reliability, scalability, and maintainability aren't independent. Optimizing one often costs another — fast iteration can introduce bugs, scaling out adds operational complexity. Every architecture is a negotiation between the three.

**Reliability** is about faults, not failures. A fault is one component deviating from spec; a failure is the whole system going down. Hardware faults are random and independent — redundancy handles them well. Software and human faults correlate and cascade, making them far harder to manage.

**Scalability** starts with describing load — not just "requests per second" but the specific parameters that matter for your system (read/write ratio, fan-out factor, concurrent users). Once you've described load, you ask: what happens to performance as it grows? Percentiles matter more than averages here — p99 tells you what your worst-off users experience, and in fan-out architectures the slowest backend call sets the pace. Vertical vs horizontal scaling has no universal answer; it depends entirely on load shape.

**Maintainability** breaks into operability (easy to run), simplicity (good abstractions, minimal accidental complexity), and evolvability (easy to change). Requirements will change — the system should accommodate that without rewrites.

## OLTP vs OLAP

The access pattern defines the workload, not the audience. OLTP is many small reads/writes with low latency (user-facing apps, DynamoDB). OLAP is fewer large scans and aggregations where seconds-to-minutes latency is acceptable (analytics dashboards, Datadog aggregating APM spans). A user-facing dashboard aggregating millions of rows is OLAP even though an end user sees it.

## Open Questions

- How does 2e handle serverless and edge compute in the scalability discussion? Traditional vertical/horizontal framing doesn't map cleanly to Lambda.
- The OLTP/OLAP boundary is blurring with real-time analytics and HTAP databases. Does 2e address this convergence?
