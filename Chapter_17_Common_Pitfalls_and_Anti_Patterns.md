# Chapter 17 -- Common Pitfalls and Anti-Patterns

## Learning Objectives

-   Identify common partitioning mistakes
-   Understand when partitioning should be avoided
-   Learn production best practices

------------------------------------------------------------------------

## Pitfall 1 -- Too Many Partitions

Thousands of tiny partitions increase planning overhead and
administrative complexity.

Choose practical partition sizes.

------------------------------------------------------------------------

## Pitfall 2 -- Wrong Partition Key

Partitioning by a column that is rarely filtered provides little
benefit.

Select a partition key that matches application queries.

------------------------------------------------------------------------

## Pitfall 3 -- Ignoring Partition Pruning

Queries that do not filter on the partition key often scan every
partition.

Always verify plans using:

``` sql
EXPLAIN ANALYZE
```

------------------------------------------------------------------------

## Pitfall 4 -- Missing Future Partitions

Applications fail when inserts arrive for dates without a matching
partition.

Automate partition creation.

------------------------------------------------------------------------

## Pitfall 5 -- Over-Indexing

Every additional index increases insert and update costs.

Create indexes only for real workloads.

------------------------------------------------------------------------

## Pitfall 6 -- Using DELETE Instead of Dropping Partitions

Deleting millions of rows:

-   Generates WAL
-   Creates dead tuples
-   Requires VACUUM

Dropping a partition is significantly faster.

------------------------------------------------------------------------

## Pitfall 7 -- Stale Statistics

Run:

``` sql
ANALYZE orders;
```

after large data changes to help the planner choose optimal execution
plans.

------------------------------------------------------------------------

## Best Practices

-   Keep partition counts manageable.
-   Align partition strategy with query patterns.
-   Automate maintenance.
-   Review execution plans regularly.
-   Test partitioning changes before production.

------------------------------------------------------------------------

# Chapter Summary

Partitioning is powerful, but only when implemented thoughtfully.
Choosing the correct partition key, maintaining healthy partitions,
validating pruning, and avoiding unnecessary complexity are essential
for long-term success.
