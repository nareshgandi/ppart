# Chapter 19 -- Hands-on Labs and Exercises

## Learning Objectives

-   Practice creating partitioned tables
-   Verify partition pruning
-   Manage partitions
-   Analyze execution plans
-   Reinforce production concepts

------------------------------------------------------------------------

# Lab 1 -- Create a Partitioned Table

Create a monthly partitioned orders table.

``` sql
CREATE TABLE orders
(
    order_id BIGINT,
    order_date DATE,
    amount NUMERIC
)
PARTITION BY RANGE(order_date);
```

Create January and February partitions.

------------------------------------------------------------------------

# Lab 2 -- Insert Data

Insert sample rows into different months.

Verify the destination partition.

``` sql
SELECT tableoid::regclass, *
FROM orders;
```

------------------------------------------------------------------------

# Lab 3 -- Verify Partition Pruning

``` sql
EXPLAIN ANALYZE
SELECT *
FROM orders
WHERE order_date='2025-01-15';
```

Confirm only one partition is scanned.

------------------------------------------------------------------------

# Lab 4 -- Create Indexes

``` sql
CREATE INDEX idx_orders_amount
ON orders(amount);
```

Verify index usage with `EXPLAIN ANALYZE`.

------------------------------------------------------------------------

# Lab 5 -- Attach and Detach

Detach an old partition.

``` sql
ALTER TABLE orders
DETACH PARTITION orders_2025_01;
```

Reattach it afterward.

------------------------------------------------------------------------

# Lab 6 -- Default Partition

Create a DEFAULT partition.

Insert a row that does not match any existing partition.

Observe where the row is stored.

------------------------------------------------------------------------

# Lab 7 -- Monitoring

List partitions.

``` sql
SELECT inhrelid::regclass
FROM pg_inherits
WHERE inhparent='orders'::regclass;
```

Check partition sizes using `pg_total_relation_size()`.

------------------------------------------------------------------------

# Self-Assessment Questions

1.  What is partition pruning?
2.  Why must unique constraints include the partition key?
3.  When should BRIN indexes be considered?
4.  Why is dropping a partition faster than deleting rows?
5.  What is the purpose of a DEFAULT partition?
6.  When should ATTACH PARTITION be used?

------------------------------------------------------------------------

# Chapter Summary

These labs reinforce the concepts covered throughout the handbook.
Completing them provides practical experience with partition creation,
pruning, indexing, maintenance, monitoring, and advanced partition
management.
