# Chapter 22 -- Managing Range Partitions with pg_partman

## Learning Objectives

-   Create a partition set with pg_partman
-   Configure automatic partition creation
-   Understand premake
-   Configure retention
-   Execute maintenance

------------------------------------------------------------------------

# Create a Parent Table

``` sql
CREATE TABLE sales
(
    sale_id BIGINT,
    sale_date DATE,
    amount NUMERIC
)
PARTITION BY RANGE (sale_date);
```

------------------------------------------------------------------------

# Register the Table

Use `create_parent()` to register the partitioned table.

Example:

``` sql
SELECT partman.create_parent(
    p_parent_table := 'public.sales',
    p_control := 'sale_date',
    p_type := 'range',
    p_interval := 'monthly'
);
```

The function creates the initial partition set and records metadata.

------------------------------------------------------------------------

# Premake

Premake determines how many future partitions pg_partman keeps
available.

Example:

``` sql
UPDATE partman.part_config
SET premake = 4
WHERE parent_table = 'public.sales';
```

This keeps four future partitions ready for incoming data.

------------------------------------------------------------------------

# Running Maintenance

Execute maintenance manually:

``` sql
SELECT partman.run_maintenance();
```

Maintenance performs tasks such as:

-   Creating future partitions
-   Applying retention
-   Updating metadata

------------------------------------------------------------------------

# Configuring Retention

Example:

``` sql
UPDATE partman.part_config
SET retention = '12 months',
    retention_keep_table = true
WHERE parent_table = 'public.sales';
```

Depending on configuration, old partitions may be detached or dropped.

------------------------------------------------------------------------

# Monitoring

List managed partition sets:

``` sql
SELECT *
FROM partman.part_config;
```

Check for rows in the DEFAULT partition:

``` sql
SELECT *
FROM partman.check_default(false);
```

List partitions:

``` sql
SELECT *
FROM partman.show_partitions('public.sales');
```

------------------------------------------------------------------------

# Automating Maintenance

Common approaches:

-   Background Worker
-   pg_cron
-   cron
-   Enterprise schedulers

Example with pg_cron:

``` sql
SELECT cron.schedule(
'partman-maintenance',
'0 * * * *',
$$SELECT partman.run_maintenance();$$
);
```

------------------------------------------------------------------------

# Troubleshooting

**Missing future partitions**

-   Verify `premake`
-   Run `partman.run_maintenance()`

**Rows in DEFAULT partition**

-   Check maintenance jobs
-   Verify partition interval
-   Review application timestamps

**Retention not working**

-   Verify retention settings in `partman.part_config`
-   Confirm maintenance is executing

------------------------------------------------------------------------

# Best Practices

-   Keep maintenance automated.
-   Monitor the DEFAULT partition.
-   Review `partman.part_config` regularly.
-   Test retention before enabling automatic partition removal.

------------------------------------------------------------------------

# Chapter Summary

pg_partman automates the operational lifecycle of range-partitioned
tables. By configuring `create_parent()`, `premake`, retention policies,
and scheduled maintenance, DBAs can keep partitioned tables healthy with
minimal manual effort.
