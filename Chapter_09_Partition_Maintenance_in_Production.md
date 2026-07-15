# Chapter 9 -- Partition Maintenance in Production

## Learning Objectives

By the end of this chapter, you will be able to:

-   Create future partitions
-   Use DEFAULT partitions
-   Archive old partitions
-   Detach and drop partitions safely
-   Automate partition maintenance

------------------------------------------------------------------------

## Why Maintenance Matters

Partitioning is not a one-time activity.

Production systems continuously:

-   Create new partitions
-   Archive old data
-   Drop expired partitions
-   Monitor partition growth

A healthy lifecycle keeps performance predictable.

------------------------------------------------------------------------

## Creating Future Partitions

Create partitions ahead of time.

``` sql
CREATE TABLE sales_2025_08
PARTITION OF sales
FOR VALUES FROM ('2025-08-01')
TO ('2025-09-01');
```

------------------------------------------------------------------------

## DEFAULT Partition

``` sql
CREATE TABLE sales_default
PARTITION OF sales DEFAULT;
```

A DEFAULT partition prevents insert failures when a matching partition
is missing.

Monitor it regularly.

``` sql
SELECT count(*) FROM sales_default;
```

------------------------------------------------------------------------

## Listing Partitions

``` sql
SELECT inhrelid::regclass AS partition_name
FROM pg_inherits
WHERE inhparent='sales'::regclass
ORDER BY partition_name;
```

------------------------------------------------------------------------

## Detaching a Partition

``` sql
ALTER TABLE sales
DETACH PARTITION sales_2025_01;
```

The detached table becomes an independent table that can be backed up or
archived.

------------------------------------------------------------------------

## Backing Up

``` bash
pg_dump -t sales_2025_01 mydb > sales_2025_01.sql
```

------------------------------------------------------------------------

## Dropping Old Partitions

``` sql
DROP TABLE sales_2025_01;
```

Dropping a partition is much faster than deleting millions of rows.

------------------------------------------------------------------------

## DELETE vs DROP

`DELETE`

-   Generates WAL
-   Creates dead tuples
-   Requires VACUUM

`DROP TABLE`

-   Fast
-   Minimal WAL
-   No dead tuples
-   No VACUUM

------------------------------------------------------------------------

## Automation

Typical monthly workflow:

1.  Create next month's partition.
2.  Verify creation.
3.  Archive expired partitions.
4.  Drop obsolete partitions.
5.  Send notification.

Automate this using cron, pg_cron, Ansible, or orchestration tools.

------------------------------------------------------------------------

# Chapter Summary

Partition maintenance is essential in production. Creating future
partitions, monitoring DEFAULT partitions, archiving historical data,
and dropping expired partitions ensures stable performance, manageable
storage growth, and predictable database operations.
