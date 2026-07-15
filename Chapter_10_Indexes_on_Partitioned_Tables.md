# Chapter 10 -- Indexes on Partitioned Tables

## Learning Objectives

-   Understand how indexes work with partitioned tables
-   Create indexes on partitioned tables
-   Learn local indexes and global index limitations
-   Design primary keys and unique constraints
-   Verify index usage with `EXPLAIN`

------------------------------------------------------------------------

## Why Indexes Still Matter

Partitioning reduces the amount of data PostgreSQL scans.

Indexes reduce the time required to find rows **within** a partition.

    Partition Pruning
            ↓
    Choose the correct partition

    Index Scan
            ↓
    Find the matching rows

Partitioning does **not** replace indexing.

------------------------------------------------------------------------

## Creating an Index

``` sql
CREATE INDEX idx_orders_customer
ON orders(customer_id);
```

PostgreSQL creates a corresponding local index on every existing
partition.

------------------------------------------------------------------------

## Local Indexes

Each partition owns its own index.

Advantages:

-   Smaller indexes
-   Faster maintenance
-   Independent rebuilds
-   Faster detach/drop operations

------------------------------------------------------------------------

## Verifying Index Usage

``` sql
EXPLAIN ANALYZE
SELECT *
FROM orders
WHERE order_date='2025-07-10'
AND customer_id=500;
```

Typical plan:

``` text
Index Scan on orders_2025_07
```

Only one partition and one index are used.

------------------------------------------------------------------------

## Unique Constraints

Unique constraints must include the partition key.

Correct:

``` sql
PRIMARY KEY (order_id, order_date)
```

Incorrect:

``` sql
PRIMARY KEY (order_id)
```

------------------------------------------------------------------------

## Global Indexes

PostgreSQL currently supports **local indexes**, not global indexes.

Trade-off:

-   Easier maintenance
-   Faster partition operations
-   Global uniqueness requires including the partition key

------------------------------------------------------------------------

## BRIN vs B-tree

**B-tree**

-   Equality lookups
-   Joins
-   Primary keys

**BRIN**

-   Large append-only partitions
-   Time-series data
-   Date range queries

------------------------------------------------------------------------

## Best Practices

-   Partition first, then index.
-   Keep only useful indexes.
-   Verify with `EXPLAIN ANALYZE`.
-   Include the partition key in unique constraints.
-   Consider BRIN for very large partitions.

------------------------------------------------------------------------

# Chapter Summary

Partitioning reduces the search space while indexes accelerate lookups
inside individual partitions. PostgreSQL automatically creates local
indexes on partitions and requires the partition key in unique
constraints because global indexes are not supported.
