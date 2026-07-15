# PostgreSQL Advanced Mastery

# Chapter 4 – HASH Partitioning

## Learning Objectives

- Understand HASH partitioning.
- Learn MODULUS and REMAINDER.
- Know when HASH is preferable to LIST or RANGE.
- Understand its limitations.

## 1. What is HASH Partitioning?

- HASH partitioning distributes rows evenly using a hash of the partition key.
- It is useful when there is no natural range or list of values.

## 2. Typical Use Cases

- Customer IDs
- User IDs
- Tenant IDs
- IoT device IDs
- Randomly distributed keys

## 3. Architecture

```text
                emp
      PARTITION BY HASH(id)
               |
     -------------------------
     |     |      |        |
   p0     p1     p2       p3
```


## 4. Creating Parent Table

```sql
CREATE TABLE emp(
 id INT,
 sal INT
) PARTITION BY HASH(id);
```


## 5. Creating Partitions

```sql
CREATE TABLE emp_p0 PARTITION OF emp
FOR VALUES WITH (MODULUS 4, REMAINDER 0);

CREATE TABLE emp_p1 PARTITION OF emp
FOR VALUES WITH (MODULUS 4, REMAINDER 1);

CREATE TABLE emp_p2 PARTITION OF emp
FOR VALUES WITH (MODULUS 4, REMAINDER 2);

CREATE TABLE emp_p3 PARTITION OF emp
FOR VALUES WITH (MODULUS 4, REMAINDER 3);
```


## 6. Insert Demo

```sql
INSERT INTO emp
SELECT g, g*100
FROM generate_series(1,20) g;
```

- Use tableoid to see which partition each row was routed to.

## 7. Verify Routing

```sql
SELECT tableoid::regclass AS partition_name, *
FROM emp
ORDER BY id;
```


## 8. How PostgreSQL Chooses a Partition

- PostgreSQL hashes the partition key internally.
- The hash value determines the remainder.
- The remainder maps the row to one partition.

## 9. Advantages

- Excellent distribution of rows.
- No hotspots from sequential values.
- Simple to manage for evenly distributed workloads.

## 10. Limitations

- Not suitable for time-series data.
- Difficult to archive by date.
- Expanding the number of hash partitions later requires data redistribution.

## 11. HASH vs LIST vs RANGE

- LIST: Known categories.
- RANGE: Continuous values like dates.
- HASH: Even distribution without natural boundaries.

## 12. Health Checks

```sql
SELECT * FROM pg_partition_tree('emp');

SELECT tableoid::regclass,* FROM emp;
```


## Trainer Tips

- Choose HASH only when query predicates rarely filter by ranges.
- For time-series data, RANGE remains the preferred strategy.

## Common Mistakes

- Using HASH for audit or order tables.
- Assuming HASH partitioning improves every query.
- Changing MODULUS in production without planning.

## Hands-on Labs

- Create an 8-partition HASH table.
- Insert 1000 rows.
- Check row distribution per partition.
- Compare with RANGE partitioning.

## Interview Questions

- What is MODULUS?
- What is REMAINDER?
- When should HASH partitioning be used?
- What are its disadvantages?

## Summary

- HASH partitioning provides even data distribution but lacks the lifecycle advantages of RANGE partitioning.
