# Chapter 18 -- Advanced Partitioning Techniques

## Learning Objectives

-   Understand advanced partitioning strategies
-   Learn sub-partitioning concepts
-   Use ATTACH and DETACH PARTITION
-   Exchange data with minimal downtime
-   Apply advanced production practices

------------------------------------------------------------------------

## Multi-Level Partitioning

A table can be partitioned more than once.

Example:

    orders
    ├── 2025_07
    │   ├── North
    │   ├── South
    │   └── West
    └── 2025_08
        ├── North
        ├── South
        └── West

The first level partitions by month, while the second level partitions
by region.

------------------------------------------------------------------------

## Sub-Partitioning

``` sql
CREATE TABLE orders
(
    order_id BIGINT,
    region TEXT,
    order_date DATE
)
PARTITION BY RANGE(order_date);
```

A child partition can itself be partitioned.

``` sql
CREATE TABLE orders_2025_07
PARTITION OF orders
FOR VALUES FROM ('2025-07-01')
TO ('2025-08-01')
PARTITION BY LIST(region);
```

------------------------------------------------------------------------

## ATTACH PARTITION

Load data into a standalone table first.

``` sql
ALTER TABLE orders
ATTACH PARTITION orders_2025_09
FOR VALUES FROM ('2025-09-01')
TO ('2025-10-01');
```

Useful during migrations and bulk loads.

------------------------------------------------------------------------

## DETACH PARTITION

``` sql
ALTER TABLE orders
DETACH PARTITION orders_2025_01;
```

The partition becomes an independent table.

------------------------------------------------------------------------

## Bulk Loading Strategy

1.  Create standalone table.
2.  Load data using COPY.
3.  Create indexes.
4.  Analyze the table.
5.  Attach it as a partition.

This minimizes production impact.

------------------------------------------------------------------------

## Partition Exchange Pattern

A common production workflow:

-   Load data offline
-   Validate
-   Attach
-   Make immediately available

------------------------------------------------------------------------

## Best Practices

-   Use sub-partitioning only when necessary.
-   Keep partition hierarchies simple.
-   Validate data before attaching partitions.
-   Use ATTACH/DETACH instead of expensive row movement.

------------------------------------------------------------------------

# Chapter Summary

Advanced partitioning techniques such as sub-partitioning, ATTACH
PARTITION, DETACH PARTITION, and staged bulk loading help manage very
large databases with minimal downtime and operational overhead.
