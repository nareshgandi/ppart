# PostgreSQL Advanced Mastery

# Chapter 3 – RANGE Partitioning (Production Guide)

## Learning Objectives

- Understand RANGE partitioning and boundary rules.
- Master MINVALUE, MAXVALUE and DEFAULT partitions.
- Learn how to safely expand ranges in production.
- Understand why populated MAXVALUE partitions cannot be split directly.
- Apply production best practices.

## 1. What is RANGE Partitioning?

- RANGE partitioning stores rows based on continuous ranges such as dates, timestamps, IDs or salary ranges.
- Typical use cases include orders, audit logs, sensor data and financial transactions.

## 2. Creating the Parent Table

```sql
CREATE TABLE emp(
 id int,
 sal int
) PARTITION BY RANGE(id);
```


## 3. Creating Partitions

```sql
CREATE TABLE emp_1 PARTITION OF emp
FOR VALUES FROM (MINVALUE) TO (100);

CREATE TABLE emp_2 PARTITION OF emp
FOR VALUES FROM (100) TO (200);

CREATE TABLE emp_3 PARTITION OF emp
FOR VALUES FROM (200) TO (MAXVALUE);
```


## 4. Boundary Rules

- FROM is inclusive.
- TO is exclusive.
- 100 belongs to the second partition, not the first.

## 5. MINVALUE

- MINVALUE covers every value below the first explicit boundary.

## 6. MAXVALUE

- MAXVALUE covers every value greater than or equal to the lower boundary.
- It does NOT replace DEFAULT for uncovered lower ranges.

## 7. DEFAULT Partition

- If the first partition starts at 1, values like 0 or -1 fail unless MINVALUE or DEFAULT is used.

## 8. Common Scenario – Expanding MAXVALUE

```text
Existing:

200 -> MAXVALUE

Business later needs:

200-300
300-400
400-MAXVALUE

Direct ATTACH is impossible because the existing MAXVALUE partition overlaps.
```


## 9. Correct Production Workflow

- Create standalone tables.
- Copy rows from the old MAXVALUE partition.
- Detach/drop the old partition.
- Attach the new partitions with non-overlapping ranges.

## 10. Time-Series Strategy

- Create monthly or quarterly partitions ahead of time.
- Use pg_partman to automate future partition creation.
- Avoid leaving a huge MAXVALUE partition for years.

## 11. Health Checks

```sql
SELECT * FROM pg_partition_tree('emp');

SELECT tableoid::regclass,* FROM emp;

SELECT relname,
pg_size_pretty(pg_total_relation_size(oid))
FROM pg_class;
```


## 12. Best Practices

- Prefer MINVALUE for the first partition.
- Use MAXVALUE carefully.
- Monitor DEFAULT.
- Plan future ranges before they are needed.

## 13. Common Mistakes

- Thinking MAXVALUE replaces DEFAULT.
- Creating overlapping partitions.
- Splitting populated MAXVALUE partitions without moving data.

## Interview Questions

- Explain FROM/TO semantics.
- Difference between MAXVALUE and DEFAULT.
- How would you split a populated MAXVALUE partition?
- Why does PostgreSQL reject overlapping partitions?

## Hands-on Labs

- Create quarterly partitions.
- Insert boundary values 99,100,199,200.
- Expand a MAXVALUE partition into three ranges.
- Verify routing with tableoid.

## Summary

- You now understand RANGE partitioning from both a learning and production perspective. The next chapter covers HASH partitioning.
