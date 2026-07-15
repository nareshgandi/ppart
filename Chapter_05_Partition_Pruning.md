# PostgreSQL Advanced Mastery
# Chapter 5 – Partition Pruning

## Learning Objectives

- Understand what partition pruning is.
- Learn static and runtime pruning.
- Read EXPLAIN plans.
- Identify situations where pruning does not occur.
- Follow production best practices.

---

## What is Partition Pruning?

Partition pruning is an optimization where PostgreSQL eliminates partitions that cannot contain matching rows.

Instead of scanning every partition, the planner scans only the required partitions.

### Example

```
orders
├── orders_2024
├── orders_2025
└── orders_2026

Query:
SELECT * FROM orders
WHERE order_date='2025-06-10';

Only orders_2025 is scanned.
```

---

## Why is it Important?

Without pruning

```
Query
  |
Scan all partitions
```

With pruning

```
Query
  |
Planner
  |
Eliminate unnecessary partitions
  |
Scan only matching partitions
```

Benefits

- Less I/O
- Faster execution
- Lower CPU
- Better cache utilization

---

## Demo Table

```sql
CREATE TABLE orders(
    id bigint,
    order_date date,
    amount numeric
) PARTITION BY RANGE(order_date);

CREATE TABLE orders_2024
PARTITION OF orders
FOR VALUES FROM ('2024-01-01') TO ('2025-01-01');

CREATE TABLE orders_2025
PARTITION OF orders
FOR VALUES FROM ('2025-01-01') TO ('2026-01-01');

CREATE TABLE orders_2026
PARTITION OF orders
FOR VALUES FROM ('2026-01-01') TO ('2027-01-01');
```

---

## Static Partition Pruning

Planner knows the constant value during planning.

```sql
EXPLAIN
SELECT *
FROM orders
WHERE order_date='2025-05-10';
```

Expected plan

```
Seq Scan on orders_2025
```

Only one partition is scanned.

---

## Runtime Partition Pruning

Planner cannot determine the value until execution.

Example

```sql
PREPARE q(date) AS
SELECT *
FROM orders
WHERE order_date=$1;

EXECUTE q('2025-05-10');
```

Modern PostgreSQL performs runtime pruning during execution.

---

## No Pruning Example

```sql
EXPLAIN
SELECT *
FROM orders;
```

Every partition is scanned because there is no filter.

---

## Another No-Pruning Example

```sql
SELECT *
FROM orders
WHERE extract(year FROM order_date)=2025;
```

The planner may not be able to prune efficiently because the partition key is wrapped inside a function.

Prefer:

```sql
WHERE order_date >= '2025-01-01'
AND order_date < '2026-01-01';
```

---

## Verify Using EXPLAIN ANALYZE

```sql
EXPLAIN ANALYZE
SELECT *
FROM orders
WHERE order_date='2025-03-15';
```

Look for only one child partition in the plan.

---

## Production Tips

- Filter using the partition key.
- Avoid unnecessary functions on the partition key.
- Create meaningful partition boundaries.
- Verify pruning using EXPLAIN.

---

## Common Mistakes

- Expecting pruning without filtering on the partition key.
- Using expressions that hide the partition key.
- Creating too many tiny partitions.

---

## Health Check

```sql
EXPLAIN
SELECT *
FROM orders
WHERE order_date='2025-01-15';
```

If multiple partitions appear unexpectedly, investigate the predicate.

---

## Interview Questions

1. What is partition pruning?
2. Difference between static and runtime pruning?
3. How do you verify pruning?
4. Why can functions prevent pruning?
5. Does pruning replace indexing?

---

## Summary

Partition pruning is the primary performance benefit of declarative partitioning. Always validate pruning using EXPLAIN or EXPLAIN ANALYZE.
