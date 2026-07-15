# Chapter 8 -- Partition Pruning

## Learning Objectives

By the end of this chapter, you will be able to:

-   Understand what partition pruning is
-   Explain why partition pruning makes queries faster
-   Verify partition pruning using `EXPLAIN`
-   Identify situations where pruning does not occur
-   Write pruning-friendly SQL queries
-   Troubleshoot common partition pruning issues

------------------------------------------------------------------------

## What is Partition Pruning?

Partition pruning is PostgreSQL's ability to **skip scanning unnecessary
partitions**.

Imagine a partitioned table with one partition for each month.

``` text
orders
├── orders_2025_01
├── orders_2025_02
├── orders_2025_03
├── orders_2025_04
├── orders_2025_05
└── orders_2025_06
```

When a query filters on the partition key:

``` sql
SELECT *
FROM orders
WHERE order_date = '2025-04-15';
```

PostgreSQL scans only the matching partition (`orders_2025_04`) and
skips the rest.

This optimization is called **partition pruning**.

------------------------------------------------------------------------

## Why It Matters

Suppose each partition contains 100 million rows and you have 24 monthly
partitions.

Without pruning:

-   Scan 24 partitions
-   Read 2.4 billion rows

With pruning:

-   Scan 1 partition
-   Read only 100 million rows

Benefits:

-   Lower I/O
-   Lower CPU usage
-   Reduced memory consumption
-   Faster execution

------------------------------------------------------------------------

## Example Table

``` sql
CREATE TABLE orders
(
    order_id BIGINT,
    order_date DATE,
    amount NUMERIC
)
PARTITION BY RANGE(order_date);
```

``` sql
CREATE TABLE orders_2025_02
PARTITION OF orders
FOR VALUES FROM ('2025-02-01')
TO ('2025-03-01');
```

------------------------------------------------------------------------

## Verifying Pruning

``` sql
EXPLAIN ANALYZE
SELECT *
FROM orders
WHERE order_date = '2025-02-15';
```

Typical plan:

``` text
Seq Scan on orders_2025_02
```

Only one partition appears in the execution plan.

------------------------------------------------------------------------

## Queries That Prevent Pruning

Avoid applying functions to the partition key.

Good:

``` sql
WHERE order_date = DATE '2025-03-10'
```

Poor:

``` sql
WHERE EXTRACT(YEAR FROM order_date)=2025
```

------------------------------------------------------------------------

## Runtime Partition Pruning

Prepared statements can still benefit from pruning.

``` sql
PREPARE get_orders(date) AS
SELECT *
FROM orders
WHERE order_date = $1;

EXECUTE get_orders('2025-03-15');
```

PostgreSQL prunes partitions at execution time.

------------------------------------------------------------------------

## Best Practices

-   Filter on the partition key.
-   Keep predicates simple.
-   Verify with `EXPLAIN`.
-   Avoid unnecessary functions.
-   Choose a partition key that matches query patterns.

------------------------------------------------------------------------

# Chapter Summary

Partition pruning is one of PostgreSQL's most important partitioning
optimizations. By skipping partitions that cannot contain matching rows,
PostgreSQL dramatically reduces I/O and improves query performance on
very large tables.
