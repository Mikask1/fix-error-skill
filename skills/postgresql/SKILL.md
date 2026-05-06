---
name: postgresql
description: Read-only queries against the PostgreSQL database — inspect schemas, query data, explain query performance, and check admin stats. Use when the user wants to inspect the database, debug data issues, or investigate schema structure. Does NOT write, update, delete, or migrate data.
---

# PostgreSQL (Read-Only)

Inspect and query the PostgreSQL database. **This skill is read-only — no INSERT, UPDATE, DELETE, DROP, TRUNCATE, ALTER, or DDL of any kind.**

## Connection

Read from environment variables (never hardcode):

```
PGHOST       — host (default: localhost)
PGPORT       — port (default: 5432)
PGDATABASE   — database name
PGUSER       — username
PGPASSWORD   — password
DATABASE_URL — full connection string (overrides above if present)
```

Connect:
```bash
psql "$DATABASE_URL"
# or
psql -h $PGHOST -p $PGPORT -U $PGUSER -d $PGDATABASE
```

---

## Allowed Operations (SELECT only)

### Inspect schema

```sql
-- List all tables
\dt

-- Describe a table
\d table_name

-- List all schemas
\dn

-- Show indexes on a table
\di table_name

-- Show foreign keys
SELECT
  tc.table_name, kcu.column_name,
  ccu.table_name AS foreign_table,
  ccu.column_name AS foreign_column
FROM information_schema.table_constraints tc
JOIN information_schema.key_column_usage kcu USING (constraint_name)
JOIN information_schema.constraint_column_usage ccu USING (constraint_name)
WHERE tc.constraint_type = 'FOREIGN KEY';

-- List columns and types
SELECT column_name, data_type, is_nullable, column_default
FROM information_schema.columns
WHERE table_name = 'table_name'
ORDER BY ordinal_position;
```

### Query data

```sql
-- Basic select with pagination (always use LIMIT)
SELECT * FROM table_name
ORDER BY created_at DESC
LIMIT 50 OFFSET 0;

-- Count with filter
SELECT COUNT(*) FROM table_name WHERE status = 'active';

-- Join example
SELECT u.id, u.email, o.id AS order_id, o.total
FROM users u
JOIN orders o ON o.user_id = u.id
WHERE o.created_at > NOW() - INTERVAL '7 days';

-- Check for nulls in a column
SELECT COUNT(*) FROM table_name WHERE column_name IS NULL;

-- Find duplicate values
SELECT column_name, COUNT(*)
FROM table_name
GROUP BY column_name
HAVING COUNT(*) > 1;
```

### Explain query performance

```sql
EXPLAIN ANALYZE SELECT ...;
```

Look for: `Seq Scan` on large tables (may need an index), high `actual rows` vs `rows` estimates, nested loop joins on large result sets.

### Admin stats (read-only)

```sql
-- Active connections
SELECT pid, usename, application_name, state, query
FROM pg_stat_activity
WHERE state != 'idle';

-- Table sizes
SELECT relname AS table, pg_size_pretty(pg_total_relation_size(relid)) AS size
FROM pg_catalog.pg_statio_user_tables
ORDER BY pg_total_relation_size(relid) DESC;

-- Long-running queries
SELECT pid, now() - query_start AS duration, query
FROM pg_stat_activity
WHERE state = 'active' AND now() - query_start > INTERVAL '30 seconds';

-- Index usage stats
SELECT relname, indexrelname, idx_scan, idx_tup_read
FROM pg_stat_user_indexes
ORDER BY idx_scan ASC;
```

---

## Critical Rules

- **NEVER run INSERT, UPDATE, DELETE, TRUNCATE, DROP, ALTER, or any DDL/DML** — this skill is strictly read-only
- **NEVER commit connection strings or passwords to the repo**
- **ALWAYS use `LIMIT`** — never run unbounded SELECT on unknown-size tables
- **If a write operation is needed**, stop and tell the user — do not attempt it here

