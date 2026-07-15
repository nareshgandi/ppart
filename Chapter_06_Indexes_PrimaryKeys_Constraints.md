# PostgreSQL Advanced Mastery
# Chapter 6 – Indexes, Primary Keys & Constraints on Partitioned Tables

## Learning Objectives

- Understand indexes on partitioned tables.
- Learn local indexes vs parent indexes.
- Understand PRIMARY KEY limitations.
- Learn UNIQUE constraint rules.
- Understand FOREIGN KEY behavior.
- Follow production best practices.

---

## Why Indexes Still Matter

Partitioning reduces the number of partitions scanned.

Indexes reduce the number of rows scanned **inside** a partition.

Partitioning is **not** a replacement for indexing.

---

## Creating an Index on the Parent

```sql
CREATE INDEX idx_orders_customer
ON orders(customer_id);
```

PostgreSQL creates a **partitioned index** on the parent and matching indexes on all existing partitions.

New partitions inherit the definition.

---

## Verify Indexes

```sql
SELECT
    tablename,
    indexname
FROM pg_indexes
WHERE tablename LIKE 'orders%';
```

---

## Local Indexes

Each partition has its own physical index.

Benefits:

- Smaller indexes
- Faster REINDEX
- Faster VACUUM
- Better cache efficiency

---

## Primary Key Rules

Example:

```sql
CREATE TABLE orders(
    id bigint,
    order_date date,
    amount numeric,
    PRIMARY KEY(id, order_date)
) PARTITION BY RANGE(order_date);
```

The partition key **must** be part of a PRIMARY KEY or UNIQUE constraint.

The following is invalid:

```sql
PRIMARY KEY(id)
```

because uniqueness cannot be guaranteed across partitions.

---

## UNIQUE Constraints

Allowed:

```sql
UNIQUE(customer_id, order_date)
```

Not allowed:

```sql
UNIQUE(customer_id)
```

unless it contains the partition key.

---

## Foreign Keys

Modern PostgreSQL supports foreign keys involving partitioned tables.

Example:

```sql
CREATE TABLE customers(
    customer_id int PRIMARY KEY
);

ALTER TABLE orders
ADD CONSTRAINT fk_customer
FOREIGN KEY(customer_id)
REFERENCES customers(customer_id);
```

Always validate FK performance with indexes on both sides.

---

## EXPLAIN Example

```sql
EXPLAIN
SELECT *
FROM orders
WHERE order_date='2025-06-01'
  AND customer_id=100;
```

Expected:

- Partition pruning
- Index Scan inside the selected partition

---

## Useful Catalog Queries

Partitioned indexes:

```sql
SELECT *
FROM pg_indexes
WHERE schemaname='public';
```

Index usage:

```sql
SELECT
    relname,
    indexrelname,
    idx_scan
FROM pg_stat_user_indexes;
```

Index sizes:

```sql
SELECT
    relname,
    pg_size_pretty(pg_relation_size(indexrelid))
FROM pg_stat_user_indexes;
```

---

## Production Best Practices

- Index the partition key when appropriate.
- Create indexes on the parent, not each partition manually.
- Keep indexes lean.
- Monitor unused indexes.
- Reindex partitions independently if needed.

---

## Common Mistakes

- Assuming partitioning removes the need for indexes.
- Creating UNIQUE constraints without the partition key.
- Forgetting indexes on foreign key columns.

---

## Interview Questions

1. Does every partition have its own index?
2. What is a partitioned index?
3. Why must the partition key be part of a PRIMARY KEY?
4. Why can't UNIQUE(id) work on a RANGE-partitioned table by date?
5. How do you verify index usage?

---

## Summary

Partitioning and indexing complement each other. Partition pruning narrows the search to relevant partitions, while indexes efficiently locate rows within those partitions.
