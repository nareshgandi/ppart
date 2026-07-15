# Chapter 23 -- Production Administration with pg_partman

## Learning Objectives

-   Execute routine pg_partman maintenance
-   Configure automated maintenance
-   Monitor partition health
-   Manage retention safely
-   Troubleshoot common production issues

------------------------------------------------------------------------

# Running Maintenance

The core maintenance function is:

``` sql
SELECT partman.run_maintenance();
```

It can:

-   Create future partitions
-   Apply retention rules
-   Maintain partition metadata

------------------------------------------------------------------------

# Automating Maintenance

Common scheduling options:

-   pg_partman Background Worker (BGW)
-   pg_cron
-   cron
-   Enterprise schedulers

Example using pg_cron:

``` sql
SELECT cron.schedule(
'pg_partman-hourly',
'0 * * * *',
$$SELECT partman.run_maintenance();$$
);
```

------------------------------------------------------------------------

# Monitoring Configuration

View managed partition sets:

``` sql
SELECT *
FROM partman.part_config;
```

Review:

-   premake
-   retention
-   automatic_maintenance
-   partition_interval

------------------------------------------------------------------------

# Monitoring DEFAULT Partitions

``` sql
SELECT *
FROM partman.check_default(false);
```

Rows returned may indicate:

-   Missing future partitions
-   Incorrect timestamps
-   Maintenance failures

------------------------------------------------------------------------

# Retention

Example:

``` sql
UPDATE partman.part_config
SET retention='12 months',
    retention_keep_table=true
WHERE parent_table='public.sales';
```

Always validate retention in a non-production environment first.

------------------------------------------------------------------------

# Common Problems

### Missing partitions

-   Verify `premake`
-   Run `partman.run_maintenance()`

### Retention not applied

-   Confirm maintenance is scheduled
-   Review `partman.part_config`

### Inserts reaching DEFAULT partition

-   Check maintenance history
-   Verify partition boundaries
-   Validate application timestamps

------------------------------------------------------------------------

# Best Practices

-   Schedule maintenance automatically.
-   Monitor maintenance jobs.
-   Keep adequate future partitions.
-   Review configuration after upgrades.
-   Test retention before enabling automatic cleanup.

------------------------------------------------------------------------

# Chapter Summary

Reliable pg_partman administration depends on automated maintenance,
proactive monitoring, and carefully tested retention policies. A healthy
maintenance routine prevents insert failures and simplifies long-term
partition lifecycle management.
