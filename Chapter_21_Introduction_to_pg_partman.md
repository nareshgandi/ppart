# Chapter 21 -- Introduction to pg_partman

## Learning Objectives

-   Understand what pg_partman is
-   Learn why it is used in production
-   Install and configure pg_partman
-   Understand its architecture
-   Compare manual partitioning with automated partition management

------------------------------------------------------------------------

# What is pg_partman?

`pg_partman` is a PostgreSQL extension that automates the creation,
maintenance, and retention of partitioned tables.

Instead of manually creating future partitions and removing old ones,
pg_partman performs these tasks automatically.

------------------------------------------------------------------------

# Why Use pg_partman?

Without automation, a DBA must:

-   Create future partitions
-   Drop expired partitions
-   Monitor missing partitions
-   Schedule maintenance jobs

pg_partman automates these activities.

Benefits:

-   Reduced operational effort
-   Fewer production incidents
-   Consistent partition maintenance
-   Easier retention management

------------------------------------------------------------------------

# Manual vs pg_partman

  Manual Partitioning         pg_partman
  --------------------------- --------------------------------
  Manual partition creation   Automatic partition creation
  Manual retention            Configurable retention policy
  DBA scripts                 Built-in maintenance functions
  Higher operational risk     Production-ready automation

------------------------------------------------------------------------

# Architecture

``` text
Application
      │
      ▼
Partitioned Parent Table
      │
      ▼
pg_partman Metadata
      │
      ▼
run_maintenance()
      │
      ├── Create future partitions
      ├── Apply retention
      └── Maintain partition sets
```

------------------------------------------------------------------------

# Installation

Install the package for your operating system, then enable it in the
target database.

``` sql
CREATE EXTENSION pg_partman;
```

Verify:

``` sql
SELECT extname, extversion
FROM pg_extension
WHERE extname='pg_partman';
```

------------------------------------------------------------------------

# Important Objects

Common objects created by the extension include:

-   `partman.part_config`
-   `partman.part_config_sub`
-   `partman.run_maintenance()`
-   `partman.show_partitions()`
-   `partman.check_default()`

------------------------------------------------------------------------

# part_config

`partman.part_config` stores configuration for every managed partition
set.

Typical information:

-   Parent table
-   Partition interval
-   Premake count
-   Retention policy
-   Automatic maintenance settings

------------------------------------------------------------------------

# Background Worker

pg_partman supports a background worker (BGW) that can periodically
execute maintenance without external schedulers.

Alternatively, many environments schedule:

-   cron
-   pg_cron
-   enterprise schedulers

------------------------------------------------------------------------

# Best Practices

-   Use native PostgreSQL partitioning.
-   Keep the premake value large enough to avoid missing partitions.
-   Monitor maintenance jobs.
-   Test retention in non-production first.

------------------------------------------------------------------------

# Chapter Summary

pg_partman extends PostgreSQL declarative partitioning by automating
partition creation, retention, and lifecycle management. It
significantly reduces DBA effort and is widely used in production
environments that manage large time-based datasets.
