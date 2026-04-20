# Fan-Out: A Primer

How Twitter's home timeline architecture illustrates the fan-out tradeoff. Based on the example from DDIA 2e, Chapter 2 (Scalability).

---

## The Problem

Twitter has two core operations:

| Operation | Volume |
|-----------|--------|
| **Post tweet** | ~5k writes/sec (peak ~12k) |
| **Load timeline** | ~300k reads/sec |

Reads outnumber writes 60:1. The architecture must optimize for reads.

---

## The Data Model

**Users:**

| user_id | handle | follower_count |
|---------|--------|---------------|
| 1 | [@seanplusplus](https://x.com/seanplusplus) | 500 |
| 2 | [@kevinrocci](https://x.com/kevinrocci) | 500 |
| 3 | [@surfkt](https://x.com/surfkt) | 500 |
| 4 | barackobama | 30,000,000 |
| 5 | elonmusk | 25,000,000 |

**Follows:**

| follower_id | follows_id | |
|-------------|-----------|---|
| 1 | 2 | seanplusplus → kevinrocci |
| 1 | 3 | seanplusplus → surfkt |
| 1 | 4 | seanplusplus → barackobama |
| 2 | 1 | kevinrocci → seanplusplus |
| 2 | 3 | kevinrocci → surfkt |
| 2 | 4 | kevinrocci → barackobama |
| 2 | 5 | kevinrocci → elonmusk |
| 3 | 1 | surfkt → seanplusplus |

**Tweets** (Postgres — source of truth for all tweets):

| tweet_id | user_id | text | created_at |
|----------|---------|------|------------|
| 101 | 2 | "surf was sick today" | 10:00 |
| 102 | 4 | "democracy is important" | 10:05 |
| 103 | 3 | "heading to trestles" | 10:10 |
| 104 | 1 | "hello world" | 10:15 |
| 105 | 5 | "posting from mars" | 10:20 |

---

## The Storage Split

| System | Stores | Role |
|--------|--------|------|
| **Postgres** | users, follows, tweets | Source of truth |
| **Redis** | timeline_cache | Pre-computed, denormalized read optimization |

Redis is **derived** — if it dies, rebuild from Postgres. It's not a source of truth, it's a read shortcut.

---

## Two Fan-Out Strategies

### Fan-Out on Read

When a user loads their timeline, query for all tweets from everyone they follow at that moment.

```sql
SELECT * FROM tweets
WHERE user_id IN (
  SELECT follows_id FROM follows WHERE follower_id = 2
)
AND created_at > now() - interval '24 hours'
ORDER BY created_at DESC
LIMIT 100
```

- **Write is cheap** — just insert the tweet
- **Read is expensive** — join across follows + tweets for every timeline load
- At 300k reads/sec, this query runs 300k times per second. Crushed.

### Fan-Out on Write

When a user tweets, push that tweet into every follower's pre-computed timeline cache in Redis.

- **Write is expensive** — one tweet triggers N cache inserts (one per follower)
- **Read is trivial** — just read the pre-computed cache
- At 300k reads/sec, each read is a simple key lookup. Fast.

---

## The Hybrid: Why Both

Fan-out on write breaks for celebrities. seanplusplus tweets → 500 cache inserts. Obama tweets → 30,000,000 cache inserts. That's a thundering herd on a single event.

Solution: **bifurcate the write path based on follower count.**

| Who tweets | follower_count | Strategy | Cost |
|------------|---------------|----------|------|
| seanplusplus | 500 | Fan-out on write | 500 Redis inserts |
| kevinrocci | 500 | Fan-out on write | 500 Redis inserts |
| surfkt | 500 | Fan-out on write | 500 Redis inserts |
| barackobama | 30,000,000 | **Skip fan-out** | 1 Postgres insert only |
| elonmusk | 25,000,000 | **Skip fan-out** | 1 Postgres insert only |

---

## The Write Path

### Normal user: seanplusplus tweets

```sql
-- 1. Insert the tweet (always happens)
INSERT INTO tweets (tweet_id, user_id, text, created_at)
VALUES (104, 1, 'hello world', now());

-- 2. Am I a celebrity?
SELECT follower_count FROM users WHERE user_id = 1;
-- Returns: 500 → nope

-- 3. Get my followers
SELECT follower_id FROM follows WHERE follows_id = 1;
-- Returns: [2, 3]

-- 4. Fan out to Redis
PUSH to timeline_cache for user_id 2: tweet_id 104
PUSH to timeline_cache for user_id 3: tweet_id 104
-- ... and the other 498 followers
```

### Celebrity: barackobama tweets

```sql
-- 1. Insert the tweet (always happens)
INSERT INTO tweets (tweet_id, user_id, text, created_at)
VALUES (102, 4, 'democracy is important', now());

-- 2. Am I a celebrity?
SELECT follower_count FROM users WHERE user_id = 4;
-- Returns: 30,000,000 → yes, skip fan-out

-- Done. No Redis writes.
```

The follower count threshold is the fork in the road. Same check, opposite outcomes.

---

## The Read Path

### kevinrocci opens his timeline

kevinrocci follows: seanplusplus (normal), surfkt (normal), barackobama (celeb), elonmusk (celeb).

**Step 1: Read Redis** — get pre-computed tweets from normal users:

```
GET timeline_cache for user_id = 2
-- Returns tweet_ids: [101 (kevinrocci? no, this is surfkt's), 103, 104, ...]
-- All normal-user tweets are already here
```

The timeline cache for kevinrocci (user_id 2) contains:

| user_id | tweet_id | created_at |
|---------|----------|------------|
| 2 | 103 | 10:10 |
| 2 | 104 | 10:15 |

These were pushed here when surfkt and seanplusplus tweeted. Waiting and ready.

**Step 2: Find celebrities kevinrocci follows:**

```sql
SELECT follows_id FROM follows
WHERE follower_id = 2
AND follows_id IN (
  SELECT user_id FROM users WHERE follower_count > 1000000
)
-- Returns: [4, 5]  (barackobama, elonmusk)
```

**Step 3: Get celebrity tweets from Postgres:**

```sql
SELECT * FROM tweets
WHERE user_id IN (4, 5)
AND created_at > now() - interval '24 hours'
ORDER BY created_at DESC
LIMIT 100
-- Returns: tweet_id 102 ("democracy is important"), tweet_id 105 ("posting from mars")
```

This query is cheap — just 2 users, not 500.

**Step 4: Merge and sort:**

| tweet_id | user_id | text | created_at |
|----------|---------|------|------------|
| 101 | 2 | "surf was sick today" | 10:00 |
| 102 | 4 | "democracy is important" | 10:05 |
| 103 | 3 | "heading to trestles" | 10:10 |
| 104 | 1 | "hello world" | 10:15 |
| 105 | 5 | "posting from mars" | 10:20 |

Timeline rendered. Two sources, one feed.

---

## The Tradeoff Summary

| | Fan-out on write | Fan-out on read |
|---|---|---|
| **Write cost** | O(followers) | O(1) |
| **Read cost** | O(1) key lookup + O(N) items | O(follows) join + sort |
| **Best for** | Users with bounded follower counts | Celebrities with massive follower counts |
| **Pain point** | Thundering herd on celebrity tweet | Expensive query on every timeline load |

The hybrid pushes write cost to the cheap side (normal users) and read cost to the rare side (celebrity tweets fetched on demand). Read-heavy systems should optimize for reads — that's the core insight.

---

## Key Takeaways

1. **Load shape drives architecture** — Twitter's challenge was the fan-out shape of the follow graph, not raw volume.
2. **Denormalize for reads** — the timeline cache is redundant data that exists purely to make reads fast.
3. **Redis is derived, Postgres is truth** — if the cache dies, rebuild it. Never the other way around.
4. **The threshold is the fork** — a single follower_count check determines the entire write path for a tweet.
5. **No universal answer** — the right strategy depends on who's posting. That's why it's a hybrid.
