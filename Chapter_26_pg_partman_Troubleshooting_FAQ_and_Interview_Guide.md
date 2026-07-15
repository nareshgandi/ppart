# Chapter 26 -- pg_partman Troubleshooting, FAQ, and Interview Guide

## Learning Objectives

-   Troubleshoot common pg_partman issues
-   Answer interview questions confidently
-   Build a production support checklist

------------------------------------------------------------------------

# Troubleshooting Guide

## Future partitions are missing

Possible causes:

-   `run_maintenance()` not running
-   `premake` too small

Check:

``` sql
SELECT *
FROM partman.part_config;
```

Run:

``` sql
SELECT partman.run_maintenance();
```

------------------------------------------------------------------------

## Inserts go to the DEFAULT partition

Verify:

-   Partition interval
-   Application timestamps
-   Maintenance schedule

``` sql
SELECT *
FROM partman.check_default(false);
```

------------------------------------------------------------------------

## Retention Is Not Working

Review:

-   `retention`
-   `retention_keep_table`
-   Scheduled maintenance

------------------------------------------------------------------------

# Frequently Asked Questions

### What problem does pg_partman solve?

It automates partition lifecycle management by creating future
partitions and applying retention policies.

### Does pg_partman replace PostgreSQL partitioning?

No. It automates native PostgreSQL partitioning.

### Can pg_partman use cron?

Yes. It supports:

-   Background Worker
-   pg_cron
-   cron
-   Enterprise schedulers

### What is `premake`?

The number of future partitions maintained ahead of current data.

### Why monitor the DEFAULT partition?

Rows in the DEFAULT partition often indicate missing partitions or
application issues.

------------------------------------------------------------------------

# Interview Questions

1.  What is pg_partman?
2.  Why use pg_partman instead of manual partition management?
3.  What does `run_maintenance()` do?
4.  What is `premake`?
5.  How is retention configured?
6.  How do you monitor managed partition sets?
7.  How do you automate maintenance?
8.  What causes rows to reach the DEFAULT partition?
9.  What is the purpose of `part_config`?
10. How would you migrate an existing partitioned table to pg_partman?

------------------------------------------------------------------------

# Final Production Checklist

-   pg_partman installed
-   Parent tables registered
-   Maintenance automated
-   Retention validated
-   Monitoring enabled
-   Backups tested
-   Recovery procedures documented

------------------------------------------------------------------------

# Chapter Summary

pg_partman greatly simplifies partition lifecycle management, but
production success depends on monitoring, automation, and regular
validation. Understanding its configuration, maintenance workflow, and
troubleshooting techniques enables DBAs to operate large partitioned
databases confidently.
