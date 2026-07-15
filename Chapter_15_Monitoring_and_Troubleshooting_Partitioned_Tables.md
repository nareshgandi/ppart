# Chapter 15 -- Monitoring and Troubleshooting Partitioned Tables

## Learning Objectives

-   Monitor partition health
-   Identify missing partitions
-   Detect partition growth
-   Troubleshoot common problems
-   Use PostgreSQL system catalogs

------------------------------------------------------------------------

## Listing Partitions

``` sql
SELECT inhrelid::regclass AS partition_name
FROM pg_inherits
WHERE inhparent='orders'::regclass
ORDER BY partition_name;
```

------------------------------------------------------------------------

## Partition Sizes

``` sql
SELECT
    relname,
    pg_size_pretty(pg_total_relation_size(oid))
FROM pg_class
WHERE relname LIKE 'orders_%';
```

Review partitions for unexpected growth.

------------------------------------------------------------------------

## Missing Partitions

A missing partition causes inserts to fail unless a DEFAULT partition
exists.

Typical error:

``` text
ERROR: no partition of relation found for row
```

Create future partitions before they are needed.

------------------------------------------------------------------------

## Monitor the DEFAULT Partition

``` sql
SELECT COUNT(*)
FROM orders_default;
```

Rows in the DEFAULT partition may indicate missing partitions or bad
application data.

------------------------------------------------------------------------

## Verify Query Plans

``` sql
EXPLAIN ANALYZE
SELECT *
FROM orders
WHERE order_date='2025-07-10';
```

Confirm that partition pruning occurs.

------------------------------------------------------------------------

## Useful Catalogs

-   `pg_inherits`
-   `pg_class`
-   `pg_stat_all_tables`
-   `pg_stat_user_indexes`

These catalogs help monitor storage, scans, and partition metadata.

------------------------------------------------------------------------

## Common Problems

-   Missing future partitions
-   Oversized partitions
-   Poor pruning
-   Unused indexes
-   Stale statistics

------------------------------------------------------------------------

## Best Practices

-   Automate partition creation.
-   Review partition sizes regularly.
-   Monitor query performance.
-   Analyze tables after bulk loads.
-   Archive old partitions on schedule.

------------------------------------------------------------------------

# Chapter Summary

Monitoring is essential for healthy partitioned tables. Regularly review
partition sizes, validate pruning, monitor the DEFAULT partition, and
use PostgreSQL system catalogs to identify problems before they affect
production.
