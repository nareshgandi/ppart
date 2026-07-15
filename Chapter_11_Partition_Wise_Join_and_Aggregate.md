# Chapter 11 -- Partition-Wise Join and Partition-Wise Aggregate

## Learning Objectives

-   Understand partition-wise joins
-   Understand partition-wise aggregation
-   Learn when PostgreSQL can use these optimizations
-   Analyze execution plans
-   Apply best practices

------------------------------------------------------------------------

## What is a Partition-Wise Join?

Suppose two tables are partitioned identically.

    orders
    ├── Jan
    ├── Feb
    └── Mar

    payments
    ├── Jan
    ├── Feb
    └── Mar

Instead of joining the entire tables, PostgreSQL can join:

-   Jan ↔ Jan
-   Feb ↔ Feb
-   Mar ↔ Mar

This is called a **partition-wise join**.

------------------------------------------------------------------------

## Benefits

-   Smaller joins
-   Better parallelism
-   Lower memory usage
-   Faster execution

------------------------------------------------------------------------

## Requirements

Both tables should:

-   Use the same partition strategy
-   Use compatible partition boundaries
-   Join using compatible partition keys

------------------------------------------------------------------------

## Example

``` sql
EXPLAIN ANALYZE
SELECT *
FROM orders o
JOIN payments p
  ON o.order_id = p.order_id
WHERE o.order_date BETWEEN '2025-07-01' AND '2025-07-31';
```

Partition pruning occurs first, followed by joins between matching
partitions.

------------------------------------------------------------------------

## Partition-Wise Aggregate

Aggregations can also be performed independently inside each partition.

Example:

``` sql
SELECT order_date,
       SUM(amount)
FROM orders
GROUP BY order_date;
```

Each partition computes its local aggregate before PostgreSQL combines
the final results.

Benefits:

-   Better parallel execution
-   Reduced memory usage
-   Faster aggregation on large datasets

------------------------------------------------------------------------

## Verifying with EXPLAIN

Look for plans showing individual partition scans followed by aggregate
or join operations.

------------------------------------------------------------------------

## Best Practices

-   Partition related tables identically.
-   Keep partition boundaries aligned.
-   Filter on the partition key.
-   Verify execution plans using `EXPLAIN ANALYZE`.

------------------------------------------------------------------------

# Chapter Summary

Partition-wise joins and partition-wise aggregates allow PostgreSQL to
process partitions independently instead of treating large partitioned
tables as a single object. When tables share compatible partitioning
schemes, these optimizations improve scalability, reduce resource
consumption, and increase query performance.
