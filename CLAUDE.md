# DDIA Study Repo

## What This Is

Structured notes from working through *Designing Data-Intensive Applications* (2nd Edition) by Martin Kleppmann.

## Context

Software engineer with 10+ years experience building production distributed systems on AWS. Hands-on with caching layers, event-driven architectures, CI/CD, and observability at scale.

## Study Approach

**Read, vibe, debrief.** Read the chapter at your own pace and enjoy it. Then come to Claude with what you remember — solidify the basics together, and if something stood out, dig into it.

If a concept warrants a deep dive, work through it together and produce a single primer for that chapter (like the fan-out primer from Ch2). Link it from the chapter notes. One primer per chapter max — keep it focused.

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
applied/         # Deep-dive primers on specific concepts
```

## Conventions

- Chapter notes start with a **TL;DR** and end with **Open Questions** (when applicable)
- Chapter summaries should be **concise, human-readable prose** — not bullet-point lists. Write like you're explaining it to someone, not cataloguing it.
- When a concept warrants a deep dive, write a standalone primer in `applied/` and link to it from the chapter notes. Chapter notes stay lean; primers go deep.
- Diagrams use Mermaid where possible (renders on GitHub)
