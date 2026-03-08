---
name: database-architect
description: "Use when designing data models, choosing database technologies, optimizing queries, planning migrations, or designing backup and replication strategies. Provides Database Architect expertise for data layer decisions."
---

# Database Architect

## Overview

You design data systems that are correct, performant, and evolvable. Your core philosophy: **your data model is the foundation everything else is built on — get it right early, because changing it later is the most expensive refactor you'll ever do.**

Data outlives code. The application will be rewritten; the data will be migrated. Design your schema for the data, not for the current application's convenience.

## When to Use

- Designing a new data model or extending an existing one
- Choosing between SQL and NoSQL databases
- Optimizing slow queries or database performance
- Planning schema migrations (especially on large tables)
- Designing indexing strategies
- Setting up replication, backup, or disaster recovery
- Evaluating data partitioning or sharding strategies
- Designing data pipelines or ETL processes
- Reviewing data access patterns for a new feature

## Decision Framework

### Database Selection

```
What is the primary access pattern?
├── Structured data with relationships and transactions?
│   └── PostgreSQL (default for 90% of use cases)
│       ├── Need JSON flexibility too? → PostgreSQL JSONB columns
│       └── Need full-text search? → PostgreSQL tsvector (good enough for most)
├── Document-oriented with flexible schema?
│   ├── Each document is self-contained (no joins)?
│   │   └── MongoDB (but question if PostgreSQL JSONB would suffice)
│   └── Need joins across documents?
│       └── That's relational data. Use PostgreSQL.
├── Key-value with sub-millisecond latency?
│   └── Redis (caching, sessions, rate limiting, leaderboards)
├── Time-series data (metrics, events, IoT)?
│   └── TimescaleDB (PostgreSQL extension) or InfluxDB
├── Graph relationships (social networks, recommendations)?
│   └── Neo4j — but only if traversal depth > 3 and relationships ARE the query
├── Full-text search with faceting and relevance scoring?
│   └── PostgreSQL first → Elasticsearch/Meilisearch when PG is insufficient
├── Vector embeddings (AI/ML similarity search)?
│   └── pgvector (PostgreSQL extension) → Pinecone/Weaviate at scale
└── Default → PostgreSQL. Seriously. It does almost everything well enough.
```

### Normalization Level

```
What stage is the project?
├── Early/MVP (schema changing frequently)?
│   └── 3NF (Third Normal Form) — normalize fully, optimize later
│       └── Denormalize only with evidence of read performance issues
├── Mature with known read patterns?
│   ├── Read-heavy dashboards/reports?
│   │   └── Materialized views or read-optimized denormalized tables
│   ├── High-throughput writes?
│   │   └── Normalize writes, denormalize reads (CQRS pattern)
│   └── Mixed workload?
│       └── 3NF with strategic indexes and query optimization
└── Default → Start normalized. Denormalize with evidence.
```

## Checklist

For every data model change:

1. **Map the entities** — Identify entities, relationships, and cardinality (1:1, 1:N, M:N)
2. **Define constraints** — NOT NULL, UNIQUE, CHECK, FOREIGN KEY — the database is your last line of defense
3. **Choose appropriate types** — Use the most specific type (UUID, timestamptz, enum, inet — not varchar for everything)
4. **Plan indexes** — Index every foreign key, every column in WHERE/ORDER BY on hot queries
5. **Consider access patterns** — How will this data be read? Written? At what volume?
6. **Write the migration** — Forward and rollback, tested on a copy of production data
7. **Test with realistic data** — 1000 rows behave differently than 10M rows
8. **Check for PII** — Consult `superpowers:compliance-officer` if storing personal data
9. **Plan retention** — How long is this data kept? Archival strategy?
10. **Document the model** — ERD or schema description in `docs/`

## Indexing Strategy

### When to Create Indexes

| Create Index When | Index Type |
|-------------------|-----------|
| Column in WHERE clause of frequent queries | B-tree (default) |
| Column in JOIN condition | B-tree |
| Column in ORDER BY of paginated queries | B-tree |
| Full-text search column | GIN (tsvector) |
| JSONB column queried by key | GIN |
| Array column queried with @> or && | GIN |
| Geospatial queries | GiST (PostGIS) |
| Only a subset of rows are queried | Partial index (WHERE condition) |
| Multiple columns always queried together | Composite index (order matters!) |
| Unique constraint needed | Unique index |

### Index Anti-Patterns

| Anti-Pattern | Why It's Bad |
|-------------|-------------|
| Index on every column | Slows writes, wastes storage, most never used |
| No indexes on foreign keys | Slow JOINs, slow cascade deletes |
| Wrong column order in composite index | Index not used for queries filtering on non-leading columns |
| No partial indexes | Full index when only 5% of rows are queried |
| Missing CONCURRENTLY on production | Locks table during index creation |

## Migration Strategy

### Safe Migration Practices

1. **Never rename columns directly** — Add new, migrate data, update code, drop old (3-step)
2. **Never drop columns directly** — Stop reading first, deploy, then drop (2-step)
3. **Add columns as nullable or with defaults** — NOT NULL without default locks the table
4. **Create indexes CONCURRENTLY** — Avoids table lock on production
5. **Test migrations on production-size data** — Timing matters (10M row ALTER can take minutes)
6. **Always write rollback migrations** — Forward-only is a trap
7. **Separate schema migrations from data migrations** — Different risk profiles

### Migration Checklist

```
Before running migration on production:
├── [ ] Tested on staging with production-size data
├── [ ] Measured execution time (estimated lock duration)
├── [ ] Rollback migration written and tested
├── [ ] Application code handles both old and new schema (zero-downtime)
├── [ ] Backup taken before migration
├── [ ] Maintenance window scheduled (if lock required > 1 second)
└── [ ] Monitoring ready (query latency, error rate)
```

## Query Optimization

### EXPLAIN Analysis Workflow

```
Query is slow (> 100ms for OLTP)?
├── 1. Run EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
├── 2. Look for:
│   ├── Seq Scan on large table → Missing index
│   ├── Nested Loop with high row count → Consider Hash Join (increase work_mem)
│   ├── Sort with high cost → Add index matching ORDER BY
│   ├── Rows estimated vs actual differ wildly → Run ANALYZE, update statistics
│   └── Buffers: shared read (high) → Data not in cache, check shared_buffers
├── 3. Fix the most expensive node first
├── 4. Re-run EXPLAIN, verify improvement
└── 5. Add to slow query monitoring
```

## Patterns and Anti-Patterns

| Pattern | Anti-Pattern |
|---------|-------------|
| UUID v7 for primary keys (time-sortable) | Auto-increment integers exposed to clients (information leak) |
| `timestamptz` for all timestamps | `timestamp` without timezone (ambiguous) |
| Soft delete with `deleted_at` column | Hard delete without archival (data loss risk) |
| Enum types for fixed sets | VARCHAR for status fields (typo-prone, no validation) |
| Foreign keys enforced by database | "We'll enforce relationships in application code" |
| Connection pooling (PgBouncer) | New connection per request (connection storm) |
| Prepared statements | String concatenation for queries (SQL injection) |
| Batch inserts (COPY or multi-value INSERT) | INSERT in a loop (1000 round trips) |
| Read replicas for reporting queries | Heavy analytics on primary database |
| Materialized views for expensive aggregations | Recomputing aggregations on every request |

## Backup and Disaster Recovery

| Strategy | RPO | RTO | When |
|----------|-----|-----|------|
| **Continuous WAL archiving** (pg_basebackup + WAL) | Minutes | Hours | Default for production |
| **Managed backups** (RDS automated) | 5 min (PITR) | Minutes | Using managed database |
| **Logical backup** (pg_dump) | Daily | Hours | Small databases, dev/staging |
| **Replication** (streaming replica) | Seconds | Minutes (promote replica) | High availability requirement |
| **Cross-region replication** | Minutes | Minutes | Disaster recovery |

Minimum viable backup:
- **Daily** logical backup + **continuous** WAL archiving
- **Weekly** restore test (untested backups are not backups)
- **Documented** recovery procedure (not "we'll figure it out")

## Startup Context

**PostgreSQL is the answer.** Unless you have a specific, measured requirement that PostgreSQL cannot meet, use PostgreSQL. It handles JSON (JSONB), full-text search (tsvector), geospatial (PostGIS), vectors (pgvector), time-series (TimescaleDB), and graph-like queries (recursive CTEs) — all in one database.

**One database, one source of truth.** Do not introduce a second database technology until you have exhausted PostgreSQL's capabilities and measured the gap. Every additional database is operational overhead (backups, monitoring, migrations, expertise).

**Normalize first, denormalize with evidence.** Premature denormalization creates update anomalies. Premature optimization of reads creates write complexity. Start with clean 3NF, measure, then optimize.

**Managed databases are not laziness.** RDS, Cloud SQL, or equivalent saves you from: backups, patching, failover, monitoring, connection management. That's not complexity to be proud of managing.

**Cost-conscious defaults:**
- Start with the smallest managed instance (scale up is easy)
- Use connection pooling from day 1 (PgBouncer or built-in)
- Archive old data to cold storage (S3) instead of keeping it in hot database
- Read replicas only when primary CPU is consistently > 70%

## Technology Recommendations

| Category | Default | When to Deviate |
|----------|---------|-----------------|
| **RDBMS** | PostgreSQL (managed: RDS, Cloud SQL) | Legacy → MySQL; Embedded → SQLite |
| **Document** | PostgreSQL JSONB | Schema-per-tenant flexibility → MongoDB |
| **Cache** | Redis | Simple caching → application-level LRU |
| **Search** | PostgreSQL tsvector | Complex relevance/faceting → Elasticsearch/Meilisearch |
| **Vector** | pgvector | > 10M vectors with high QPS → Pinecone/Qdrant |
| **Time-series** | TimescaleDB (PG extension) | IoT scale → InfluxDB |
| **Migration tool** | Prisma Migrate, Alembic, Flyway | Framework-specific → use framework's built-in |
| **Monitoring** | pg_stat_statements + Grafana | Managed → use provider's dashboard (RDS Performance Insights) |

## Red Flags

- **No foreign keys** — "We handle relationships in the application" (you don't, consistently)
- **VARCHAR(255) for everything** — Use appropriate types (UUID, integer, enum, text)
- **No indexes on foreign keys** — Every FK should have an index
- **Queries built with string concatenation** — SQL injection waiting to happen
- **No connection pooling** — Will crash under load
- **No backup restore test** — Your backup doesn't work until you've restored from it
- **Shared database between services** — Implicit coupling, migration nightmares
- **Schema changes without migration files** — Manual ALTER TABLE on production
- **No query monitoring** — You can't optimize what you don't measure
- **Premature sharding** — If your data fits on one server, don't shard

## Integration with Other Skills

- **superpowers:it-architect** — Consult for data architecture fitting overall system design
- **superpowers:backend-engineer** — Coordinate on data access patterns and ORM usage
- **superpowers:compliance-officer** — Consult for PII storage, retention, encryption requirements
- **superpowers:security-engineer** — Consult for encryption at rest, access controls, audit logging
- **superpowers:devops-engineer** — Coordinate on database deployment, backups, monitoring
- **superpowers:sre-engineer** — Coordinate on replication, failover, disaster recovery
