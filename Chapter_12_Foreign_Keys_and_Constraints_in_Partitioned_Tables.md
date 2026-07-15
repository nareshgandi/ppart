# Chapter 12 -- Foreign Keys and Constraints in Partitioned Tables

## Learning Objectives

-   Understand foreign keys on partitioned tables
-   Learn primary key and unique constraint rules
-   Understand CHECK, NOT NULL, and DEFAULT propagation
-   Apply production best practices

------------------------------------------------------------------------

## Foreign Keys

``` sql
ALTER TABLE orders
ADD CONSTRAINT fk_customer
FOREIGN KEY (customer_id)
REFERENCES customers(customer_id);
```

The foreign key is defined on the parent table and automatically applies
to all partitions.

------------------------------------------------------------------------

## Primary Keys

Correct:

``` sql
PRIMARY KEY (order_id, order_date)
```

Incorrect:

``` sql
PRIMARY KEY (order_id)
```

Primary keys and unique constraints must include the partition key
because PostgreSQL uses local indexes.

------------------------------------------------------------------------

## CHECK Constraints

``` sql
ALTER TABLE orders
ADD CONSTRAINT chk_amount
CHECK (amount > 0);
```

CHECK constraints are inherited by all partitions.

------------------------------------------------------------------------

## NOT NULL and DEFAULT

Definitions on the parent table automatically propagate to partitions.

------------------------------------------------------------------------

## Referencing a Partitioned Table

A foreign key can reference a partitioned table as long as it references
the parent table's primary key.

------------------------------------------------------------------------

## Best Practices

-   Define constraints on the parent table.
-   Include the partition key in unique constraints.
-   Verify constraints using `pg_constraint`.
-   Test cascading actions before production deployment.

------------------------------------------------------------------------

# Chapter Summary

Partitioned tables fully support foreign keys, CHECK constraints, NOT
NULL constraints, and DEFAULT values. Primary keys and unique
constraints must include the partition key because PostgreSQL does not
support global indexes.
