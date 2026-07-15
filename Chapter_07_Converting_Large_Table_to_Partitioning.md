# PostgreSQL Advanced Mastery
# Chapter 7 – Converting a Large Orders Table to Declarative Partitioning

## Learning Objectives

- Design a partitioning strategy for a production table.
- Create a partitioned replacement table.
- Backfill existing data.
- Validate migration.
- Perform a low-downtime cutover.

---

## Scenario

A production table named `orders_flat` has grown to hundreds of millions of rows.

Problems:

- Slow VACUUM
- Long backups
- Large indexes
- Expensive deletes
- Increasing query latency

Goal:

Convert the table to native declarative RANGE partitioning.

---

## Existing Table

```sql
CREATE TABLE orders_flat (
    id          BIGINT PRIMARY KEY,
    order_date  TIMESTAMP NOT NULL,
    customer_id INT NOT NULL,
    region      TEXT,
    amount      NUMERIC(10,2),
    details     TEXT
);
```

Choose the partition key:

- `order_date` (recommended for time-series)
- Monthly or quarterly partitions

---

## Create the New Parent Table

```sql
CREATE TABLE orders_range (
    id          BIGINT NOT NULL,
    order_date  TIMESTAMP NOT NULL,
    customer_id INT NOT NULL,
    region      TEXT,
    amount      NUMERIC(10,2),
    details     TEXT
) PARTITION BY RANGE (order_date);
```

---

## Create Initial Partitions

```sql
CREATE TABLE orders_2025_q1
PARTITION OF orders_range
FOR VALUES FROM ('2025-01-01') TO ('2025-04-01');

CREATE TABLE orders_2025_q2
PARTITION OF orders_range
FOR VALUES FROM ('2025-04-01') TO ('2025-07-01');

CREATE TABLE orders_default
PARTITION OF orders_range DEFAULT;
```

---

## Create Indexes

Always create indexes on the **parent** table.

```sql
CREATE INDEX idx_orders_customer
ON orders_range(customer_id);

CREATE INDEX idx_orders_region
ON orders_range(region);
```

---

## Backfill Existing Data

```sql
INSERT INTO orders_range
SELECT *
FROM orders_flat;
```

For very large tables, migrate in batches:

```sql
INSERT INTO orders_range
SELECT *
FROM orders_flat
WHERE id BETWEEN 1 AND 100000;
```

Repeat until complete.

---

## Validate the Migration

Compare row counts.

```sql
SELECT count(*) FROM orders_flat;

SELECT count(*) FROM orders_range;
```

Validate sample rows.

```sql
SELECT *
FROM orders_flat
EXCEPT
SELECT *
FROM orders_range;
```

---

## Verify Partition Routing

```sql
SELECT
tableoid::regclass,
count(*)
FROM orders_range
GROUP BY tableoid;
```

---

## Cutover Strategy

During a maintenance window:

```sql
BEGIN;

ALTER TABLE orders_flat
RENAME TO orders_flat_old;

ALTER TABLE orders_range
RENAME TO orders_flat;

COMMIT;
```

Applications continue using the original table name.

---

## Rollback

If validation fails:

```sql
BEGIN;

ALTER TABLE orders_flat
RENAME TO orders_failed;

ALTER TABLE orders_flat_old
RENAME TO orders_flat;

COMMIT;
```

---

## Health Checks

```sql
SELECT *
FROM pg_partition_tree('orders_flat');

SELECT
tableoid::regclass,
count(*)
FROM orders_flat
GROUP BY tableoid;
```

---

## Production Best Practices

- Choose the correct partition key.
- Create future partitions in advance.
- Use DEFAULT only as a safety net.
- Validate row counts before cutover.
- Keep the old table until business validation is complete.

---

## Common Mistakes

- Partitioning by the wrong column.
- Forgetting indexes.
- Skipping validation.
- Performing cutover without rollback planning.

---

## Interview Questions

1. How do you migrate a large table to partitioning?
2. How do you validate migrated data?
3. Why keep the old table?
4. Why create indexes on the parent?
5. How would you minimize downtime?

---

## Summary

You now understand the offline migration pattern. The next chapter builds on this approach with **online migration using dual-write triggers**, allowing continuous writes during migration.
