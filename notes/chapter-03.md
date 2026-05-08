# Chapter 3: Data Models and Query Languages

## TL;DR

A tour of how we model and query data — from relational to document to graph. Each model makes certain queries easy and others awkward. The choice of data model is the most fundamental decision in application design because it shapes how you *think* about the problem.

## The Models

**Relational** — tables, rows, joins. Schema-on-write. General-purpose workhorse since the '70s. The DB enforces structure; write wrong, get rejected.

**Document** — JSON/BSON blobs. Schema-on-read. Great for self-contained records with tree structure (one-to-many). Weak at many-to-many — you denormalize or do app-side joins. The "schema" is implicit in your parsing logic.

**Graph** — vertices and edges. When relationships *are* the data. Variable-length traversals are first-class primitives, not hacks bolted onto tabular models.

## Key Tensions

| Dimension | Relational | Document | Graph |
|-----------|-----------|----------|-------|
| Schema | On-write (static typing) | On-read (dynamic typing) | On-read |
| Joins | Native | Weak/manual | Native (traversals) |
| Data locality | Scattered across tables | Single document | Scattered |
| Many-to-many | Natural | Awkward | Natural |
| Schema evolution | Migration required | Free (app logic) | Free |

## Query Languages

- **SQL** — declarative, optimizer picks execution plan
- **MapReduce** — functional primitives (map + reduce) over distributed data
- **Cypher** — declarative graph pattern matching (Neo4j)
- **SPARQL** — RDF triple pattern matching
- **Datalog** — logic programming; rules build on rules

## Convergence

The relational/document line is blurring. Postgres has JSONB with GIN indexes. MongoDB added joins. You can have relational structure where you need integrity and document flexibility where the schema is volatile — in the same database.

## Interview Takeaways

**"JSONB preferences column"** — classic convergence play. Relational for structure (uniqueness, FKs), document for the volatile blob. Tradeoff: lose queryability (no joins into the JSON), gain locality and free schema evolution. GIN indexes claw some queryability back.

**"Fraud detection within 3 hops"** — textbook graph. A 3-hop query in SQL means self-joining 3 times or recursive CTEs; cost multiplies per hop. Graph engines walk adjacency lists natively. `MATCH (buyer)-[*1..3]-(connected) WHERE connected.flagged = true` — that's the whole query.

**Schema-on-read vs write** — structure exists in both; question is *where* it's enforced. On-write = DB is the gatekeeper. On-read = app code is responsible for interpretation. Kleppmann's analogy: static vs dynamic typing.
