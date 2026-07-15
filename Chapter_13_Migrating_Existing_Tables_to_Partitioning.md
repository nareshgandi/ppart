# Chapter 13 -- Migrating Existing Tables to Partitioning

## Learning Objectives

-   Plan a partitioning migration
-   Understand online and offline approaches
-   Move data safely
-   Validate migrated data
-   Minimize downtime

------------------------------------------------------------------------

## Why Migrate?

Large monolithic tables eventually suffer from:

-   Slow maintenance
-   Large indexes
-   Long VACUUM operations
-   Poor archival performance

Partitioning addresses these issues.

------------------------------------------------------------------------

## Migration Strategies

### Offline

1.  Stop writes.
2.  Create partitioned table.
3.  Copy data.
4.  Rename tables.
5.  Resume application.

Simple but requires downtime.

------------------------------------------------------------------------

### Online

1.  Create partitioned table.
2.  Create partitions.
3.  Copy historical data in batches.
4.  Synchronize new changes.
5.  Switch application.
6.  Validate results.

Suitable for production systems.

------------------------------------------------------------------------

## Creating the New Table

``` sql
CREATE TABLE orders_new
(
    order_id BIGINT,
    customer_id INT,
    order_date DATE,
    amount NUMERIC
)
PARTITION BY RANGE(order_date);
```

------------------------------------------------------------------------

## Moving Data

``` sql
INSERT INTO orders_new
SELECT *
FROM orders_old;
```

For very large tables, migrate in batches.

------------------------------------------------------------------------

## Validation

``` sql
SELECT COUNT(*) FROM orders_old;

SELECT COUNT(*) FROM orders_new;
```

Compare row counts and sample data.

------------------------------------------------------------------------

## Renaming Tables

``` sql
ALTER TABLE orders RENAME TO orders_old;

ALTER TABLE orders_new RENAME TO orders;
```

------------------------------------------------------------------------

## Best Practices

-   Test the migration in staging.
-   Back up the original table.
-   Migrate during low activity.
-   Validate counts and checksums.
-   Monitor application performance after cutover.

------------------------------------------------------------------------

# Chapter Summary

Migrating an existing table to partitioning requires planning, careful
data movement, validation, and a controlled cutover. Online migration
strategies minimize downtime and are recommended for production
databases.
