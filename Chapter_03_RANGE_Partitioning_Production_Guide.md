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

## NEW Content

# Chapter 3 – RANGE Partitioning

## 3.1 What is RANGE Partitioning?

RANGE partitioning stores rows based on a continuous range of values.

Typical partition keys include:

- Dates
- Timestamps
- Order IDs
- Employee IDs
- Salary ranges

Unlike LIST partitioning, RANGE partitioning stores data between a lower and upper boundary.

Typical production use cases:

- Order Management Systems
- Banking Transactions
- Audit Logs
- IoT Sensor Data
- Billing Systems
- Financial Applications

---

## 3.2 Create the Parent Table

Create a partitioned table using the `RANGE` method.

```sql
CREATE TABLE emp
(
    id  INT,
    sal INT
)
PARTITION BY RANGE(id);
```

Verify the table.

```sql
\d+ emp
```

Output:

```text
Partition key: RANGE (id)
Number of partitions: 0
```

Currently no partitions exist.

---

## 3.3 Create Three Partitions

Create three partitions.

```sql
CREATE TABLE emp_1
PARTITION OF emp
FOR VALUES FROM (MINVALUE) TO (100);

CREATE TABLE emp_2
PARTITION OF emp
FOR VALUES FROM (100) TO (200);

CREATE TABLE emp_3
PARTITION OF emp
FOR VALUES FROM (200) TO (MAXVALUE);
```

Verify.

```sql
\d+ emp
```

Output:

```text
emp_1
FROM (MINVALUE) TO (100)

emp_2
FROM (100) TO (200)

emp_3
FROM (200) TO (MAXVALUE)
```

---

## 3.4 Insert Sample Data

Insert some rows.

```sql
INSERT INTO emp VALUES
(25,1000),
(75,1500),
(100,2000),
(125,2500),
(199,3000),
(200,3500),
(250,4000),
(500,5000);
```

Verify where PostgreSQL stored each row.

```sql
SELECT
tableoid::regclass AS partition_name,
*
FROM emp
ORDER BY id;
```

Output

```text
partition_name   id   sal
-------------- ---- ------
emp_1           25   1000
emp_1           75   1500

emp_2          100   2000
emp_2          125   2500
emp_2          199   3000

emp_3          200   3500
emp_3          250   4000
emp_3          500   5000
```

Notice:

100 is stored in **emp_2**, not **emp_1**.

---

## 3.5 Understanding Boundary Rules

One of the most important concepts in RANGE partitioning is the partition boundary.

Each partition follows this rule:

```text
FROM  → Inclusive

TO    → Exclusive
```

Example

```text
emp_1

FROM MINVALUE

TO 100
```

Contains

```text
...
97

98

99
```

But NOT

```text
100
```

The value **100** belongs to

```text
emp_2
```

Similarly,

```text
emp_2

FROM 100

TO 200
```

Contains

```text
100

101

...

199
```

but NOT

```text
200
```

---

## 3.6 Understanding MINVALUE

MINVALUE represents every value smaller than the first explicit boundary.

```text
MINVALUE

↓

...

-100

-10

-1

0

1

2
```

Insert

```sql
INSERT INTO emp VALUES
(-100,100),
(-5,200),
(0,300);
```

Verify

```sql
SELECT
tableoid::regclass,
*
FROM emp
WHERE id<=0;
```

Output

```text
emp_1
```

All rows go to the first partition because it starts with MINVALUE.

---

## 3.7 Understanding MAXVALUE

MAXVALUE represents every value greater than or equal to the final lower boundary.

Insert

```sql
INSERT INTO emp VALUES
(1000,100),
(5000,200),
(999999,300);
```

Verify

```sql
SELECT
tableoid::regclass,
*
FROM emp
WHERE id>=1000;
```

Output

```text
emp_3
```

All large values go into the last partition.

---

## 3.8 Simulating a Missing Range (300–400)

Suppose your existing partitions are:

```text
emp_1

MINVALUE → 100

emp_2

100 → 200

emp_3

200 → MAXVALUE
```

Now imagine your business decides that IDs **300–399** should be stored in a dedicated partition.

A common mistake is trying to attach a new partition directly.

```sql
CREATE TABLE emp_4
PARTITION OF emp
FOR VALUES FROM (300) TO (400);
```

PostgreSQL returns:

```text
ERROR:
partition "emp_4" would overlap partition "emp_3"
```

Why?

Because the existing partition:

```text
emp_3

FROM 200

TO MAXVALUE
```

already owns the range **300–400**.

PostgreSQL does not allow overlapping partitions.

---

## 3.9 Correct Way to Split an Existing Range

To create a dedicated partition for **300–400**, follow these steps.

### Step 1 – Detach the existing partition

```sql
ALTER TABLE emp
DETACH PARTITION emp_3;

alter table emp_3 rename to old_emp_3;
```

`emp_3` is now a standalone table.

---

### Step 2 – Create new partitions

```sql
CREATE TABLE emp_3
PARTITION OF emp
FOR VALUES FROM (200) TO (300);

CREATE TABLE emp_4
PARTITION OF emp
FOR VALUES FROM (300) TO (400);

CREATE TABLE emp_5
PARTITION OF emp
FOR VALUES FROM (400) TO (MAXVALUE);
```

---

### Step 3 – Move the data

```sql
INSERT INTO emp_3
SELECT *
FROM old_emp_3
WHERE id>=200
AND id<300;

INSERT INTO emp_4
SELECT *
FROM old_emp_3
WHERE id>=300
AND id<400;

INSERT INTO emp_5
SELECT *
FROM old_emp_3
WHERE id>=400;
```

---

### Step 4 – Verify

```sql
SELECT
tableoid::regclass,
*
FROM emp
ORDER BY id;
```

Rows are now distributed correctly.

---

## 3.10 Trainer Tips

- Always define non-overlapping ranges.
- Remember **FROM is inclusive** and **TO is exclusive**.
- Use `MINVALUE` for the first partition whenever possible.
- Use `MAXVALUE` for the last partition when future growth is unknown.
- Use `tableoid::regclass` to verify where rows are physically stored.
- When changing partition boundaries, detach the existing partition, split the data, and create new partitions.

---

## Common Mistakes

- Assuming `TO` is inclusive.
- Creating overlapping partitions.
- Forgetting to verify row placement.
- Not planning future partition growth.
- Attempting to create a partition whose range is already owned by another partition.
- Expand a MAXVALUE partition into three ranges.
- Verify routing with tableoid.

## Summary

- You now understand RANGE partitioning from both a learning and production perspective. The next chapter covers HASH partitioning.
