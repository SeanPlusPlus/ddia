# DDIA Study Repo

## What This Is

Structured notes and exercises from working through *Designing Data-Intensive Applications* (2nd Edition) by Martin Kleppmann.

## Context

Sean is a Lead Software Engineer at Disney building production platforms (AWS/React/TypeScript) and autonomous AI agent systems on AWS (Lambda, ECS, CloudFront, CDK). He has hands-on experience with:

- Multi-layer caching (CloudFront, Varnish, HAProxy)
- Event-driven architectures (Lambda, SQS, EventBridge)
- CI/CD pipelines (GitLab)
- Observability (Datadog RUM/APM)
- Production distributed systems at scale

## How to Help

When discussing DDIA concepts:

- **Connect theory to practice** — relate concepts to AWS services, CDN architecture, caching layers, and real distributed systems Sean works with
- **Be precise** — use correct terminology, cite chapter/section when referencing the book
- **Challenge understanding** — ask probing questions, don't just summarize
- **Applied examples** — when a concept maps to a real system (DynamoDB = LSM-tree, CloudFront = CDN consistency model), call it out
- **No hand-holding** — Sean has production distributed systems experience, skip the basics

## Repo Structure

```
notes/           # Chapter-by-chapter reading notes
exercises/       # Worked examples and thought experiments
applied/         # Connections to real systems
```

## Conventions

- Notes use markdown with clear headers per section
- Diagrams use Mermaid where possible (renders on GitHub)
- Each chapter note starts with a **TL;DR** and ends with **Open Questions**
