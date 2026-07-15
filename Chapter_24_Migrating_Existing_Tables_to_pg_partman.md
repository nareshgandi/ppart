# Chapter 24 -- Migrating Existing Tables to pg_partman

## Learning Objectives

-   Convert an existing partitioned table to pg_partman
-   Register existing partition sets
-   Migrate large tables safely
-   Minimize production downtime
-   Validate migration results

------------------------------------------------------------------------

# Migration Strategy

Typical workflow:

1.  Back up the database.
2.  Create or verify the partitioned table.
3.  Install pg_partman.
4.  Register the parent table.
5.  Validate metadata.
6.  Schedule maintenance.

------------------------------------------------------------------------

# Existing Partitioned Table

Suppose the following table already exists:

``` sql
CREATE TABLE sales
(
    sale_id BIGINT,
    sale_date DATE,
    amount NUMERIC
)
PARTITION BY RANGE (sale_date);
```

Existing monthly partitions are already attached.

------------------------------------------------------------------------

# Register with pg_partman

Register the parent:

``` sql
SELECT partman.create_parent(
    p_parent_table := 'public.sales',
    p_control := 'sale_date',
    p_type := 'range',
    p_interval := 'monthly'
);
```

After registration, verify:

``` sql
SELECT *
FROM partman.part_config
WHERE parent_table='public.sales';
```

------------------------------------------------------------------------

# Validate Existing Partitions

List partitions:

``` sql
SELECT *
FROM partman.show_partitions('public.sales');
```

Confirm that all expected partitions are present.

------------------------------------------------------------------------

# Bulk Data Migration

For large datasets:

1.  Load historical data.
2.  Create indexes.
3.  Run `ANALYZE`.
4.  Execute `partman.run_maintenance()`.
5.  Validate row counts.

------------------------------------------------------------------------

# Cutover Checklist

-   Application paused (if required)
-   Counts validated
-   Queries tested
-   Future partitions created
-   Maintenance scheduled
-   Backups verified

------------------------------------------------------------------------

# Rollback Plan

Keep the original table or backup until:

-   Application validation completes
-   Performance is verified
-   Business acceptance testing finishes

------------------------------------------------------------------------

# Best Practices

-   Test migrations in staging first.
-   Validate every partition after migration.
-   Monitor execution plans after cutover.
-   Enable automation only after successful validation.

------------------------------------------------------------------------

# Chapter Summary

Migrating existing partitioned tables to pg_partman enables automated
lifecycle management without changing application queries. Careful
planning, validation, and staged deployment minimize operational risk
while providing long-term maintenance benefits.
