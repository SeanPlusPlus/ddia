# DDIA Study Repo

## What This Is

Structured notes and exercises from working through *Designing Data-Intensive Applications* (2nd Edition) by Martin Kleppmann.

## Context

Software engineer with 10+ years experience building production distributed systems on AWS. Hands-on with caching layers, event-driven architectures, CI/CD, and observability at scale.

## Study Approach

**Find gems, dig deep, move on.** Not trying to exhaustively cover every section — skip what's already familiar, zero in on concepts that click or challenge, and work those until the mental model is solid. Then next chapter.

Don't push for completeness. No need to cover every topic in a chapter. If the interesting stuff is done, it's done.

## How to Help

When discussing DDIA concepts:

- **Connect theory to practice** — relate concepts to real AWS services, CDN architecture, caching layers, and production distributed systems
- **Be precise** — use correct terminology, cite chapter/section when referencing the book
- **Challenge understanding** — ask probing questions, don't just summarize
- **Applied examples** — when a concept maps to a real system (DynamoDB = LSM-tree, CloudFront = CDN consistency model), call it out
- **No hand-holding** — assume production distributed systems experience, skip the basics
- **Don't nag about unfinished sections** — if the user moves on, the chapter is done

## Repo Structure

```
notes/           # Chapter-by-chapter reading notes
exercises/       # Worked examples and thought experiments
applied/         # Connections to real systems
```

## Conventions

- Chapter notes start with a **TL;DR** and end with **Open Questions** (when applicable)
- Chapter summaries should be **concise, human-readable prose** — not bullet-point lists. Write like you're explaining it to someone, not cataloguing it.
- When a concept warrants a deep dive, write a standalone primer in `applied/` and link to it from the chapter notes. Chapter notes stay lean; primers go deep.
- Diagrams use Mermaid where possible (renders on GitHub)
