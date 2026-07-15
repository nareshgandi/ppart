# Chapter 25 -- pg_partman Best Practices, Monitoring, and Performance

## Learning Objectives

-   Monitor pg_partman in production
-   Understand key configuration tables
-   Tune partition maintenance
-   Avoid common operational mistakes

------------------------------------------------------------------------

# Understanding part_config

The `partman.part_config` table stores the configuration for every
managed partition set.

Typical fields include:

-   `parent_table`
-   `partition_interval`
-   `premake`
-   `retention`
-   `automatic_maintenance`

Review configuration:

``` sql
SELECT *
FROM partman.part_config;
```

------------------------------------------------------------------------

# Monitoring Managed Tables

List all managed partition sets:

``` sql
SELECT parent_table,
       partition_interval,
       premake,
       retention
FROM partman.part_config;
```

Review this regularly after upgrades and schema changes.

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

# Performance Tips

-   Keep `premake` large enough for expected inserts.
-   Schedule `run_maintenance()` during off-peak hours when possible.
-   Run `ANALYZE` after large data loads.
-   Monitor partition growth and index size.

------------------------------------------------------------------------

# Common Mistakes

-   Forgetting to schedule maintenance
-   Setting retention too aggressively
-   Ignoring the DEFAULT partition
-   Creating excessive numbers of partitions

------------------------------------------------------------------------

# Production Checklist

-   Maintenance scheduled
-   Future partitions available
-   Retention tested
-   Backups verified
-   Monitoring dashboards configured

------------------------------------------------------------------------

# Chapter Summary

Production success with pg_partman depends on regular monitoring,
validated configuration, sensible retention policies, and automated
maintenance.
