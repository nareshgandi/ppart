# PostgreSQL Advanced Mastery
# Volume 5 – Partitioning at Production Scale

# Chapter 2 – LIST Partitioning (EMP Lab)

## Learning Objectives

By the end of this chapter you will:

- Understand LIST partitioning.
- Create LIST partitioned tables.
- Understand DEFAULT partitions.
- Learn ATTACH and DETACH.
- Move rows from DEFAULT into new partitions.
- Verify partition routing using tableoid.

---

# 2.1 What is LIST Partitioning?

LIST partitioning stores rows based on discrete values.

Examples:
- Region
- Country
- Department
- Status
- Tenant ID

Choose LIST partitioning when values belong to well-defined categories.

---

# 2.2 Creating the Parent Table

```sql
CREATE TABLE emp(
    id  INT,
    sal INT
) PARTITION BY LIST(id);
```

The parent is a virtual table.

---

# 2.3 Creating Partitions

```sql
CREATE TABLE emp_1
PARTITION OF emp
FOR VALUES IN (1,4,6);

CREATE TABLE emp_2
PARTITION OF emp
FOR VALUES IN (2,3,5);

CREATE TABLE emp_3
PARTITION OF emp
FOR VALUES IN (8,9);
```

Diagram

```text
                 emp
        PARTITION BY LIST(id)
                 |
     -------------------------
     |          |           |
   emp_1      emp_2      emp_3
 (1,4,6)    (2,3,5)     (8,9)
```

---

# 2.4 Insert Demo

```sql
INSERT INTO emp VALUES
(1,1000),
(2,2000),
(3,3000),
(4,4000),
(5,5000),
(6,6000),
(8,8000),
(9,9000);
```

Although rows are inserted into the parent, PostgreSQL stores them in the correct child partition.

---

# 2.5 Verify Routing

```sql
SELECT tableoid::regclass AS partition_name,* 
FROM emp
ORDER BY id;
```

Expected Output

```text
partition_name | id | sal
---------------+----+------
emp_1          | 1  |1000
emp_2          | 2  |2000
...
```

---

# 2.6 DEFAULT Partition

Without a DEFAULT partition:

```sql
INSERT INTO emp VALUES (7,7000);
```

Result:

```text
ERROR:
no partition found for row
```

Create DEFAULT

```sql
CREATE TABLE emp_default
PARTITION OF emp DEFAULT;
```

Now every unknown value is accepted.

---
## 2.7 Creating a New LIST Partition After Data Already Exists

One of the most common production scenarios is adding a new partition after data has already been routed to the `DEFAULT` partition.

Suppose the `emp_default` partition already contains the following row:

```text
id
----
7
```

Attempting to attach a new partition immediately:

```sql
CREATE TABLE emp_4 (LIKE emp INCLUDING ALL);

ALTER TABLE emp
ATTACH PARTITION emp_4
FOR VALUES IN (7);
```

results in:

```text
ERROR:
updated partition constraint for default partition "emp_default"
would be violated by some row
```

This happens because the value **7** already belongs to the `DEFAULT` partition. PostgreSQL cannot allow the same partition key to exist in both the new partition and the `DEFAULT` partition.

### Correct Workflow

### Step 1 – Create a standalone table

```sql
CREATE TABLE emp_4 (LIKE emp INCLUDING ALL);
```

At this stage, `emp_4` is an ordinary table and is **not** yet part of the partition hierarchy.

---

### Step 2 – Add a matching CHECK constraint

```sql
ALTER TABLE emp_4
ADD CONSTRAINT emp_4_ck
CHECK (id IS NOT NULL AND id = 7);
```

This temporary CHECK constraint tells PostgreSQL that every row in `emp_4` satisfies the intended partition boundary.

During `ATTACH PARTITION`, PostgreSQL can use this validated constraint to avoid scanning the entire table.

---

### Step 3 – Move the matching rows

```sql
BEGIN;

INSERT INTO emp_4
SELECT *
FROM emp_default
WHERE id = 7;

DELETE
FROM emp_default
WHERE id = 7;
```

Now the `DEFAULT` partition no longer contains rows with `id = 7`.

---

### Step 4 – Attach the partition

```sql
ALTER TABLE emp
ATTACH PARTITION emp_4
FOR VALUES IN (7);

COMMIT;
```

The attach operation now succeeds because there are no conflicting rows remaining in the `DEFAULT` partition.

---

### Step 5 – Remove the temporary CHECK constraint (Optional)

```sql
ALTER TABLE emp_4
DROP CONSTRAINT emp_4_ck;
```

The CHECK constraint was needed only to help PostgreSQL validate the existing data efficiently during the attach operation.

Once the partition has been attached, PostgreSQL stores the partition boundary as partition metadata. The temporary CHECK constraint is no longer required and can be safely removed.

---

### Verify the Partition

Display the partition hierarchy:

```sql
\d+ emp
```

Example output:

```text
Partition key: LIST (id)

Partitions:
    emp_1 FOR VALUES IN (1,4,6)
    emp_2 FOR VALUES IN (2,3,5)
    emp_3 FOR VALUES IN (8,9)
    emp_4 FOR VALUES IN (7)
    emp_default DEFAULT
```

Verify where the row is physically stored:

```sql
SELECT
    tableoid::regclass AS partition_name,
    *
FROM emp
WHERE id = 7;
```

Example output:

```text
 partition_name | id | sal
----------------+----+-----
 emp_4          |  7 | ...
```

This confirms that the row has been successfully moved from the `DEFAULT` partition to the newly attached partition.

---

## 2.8 CREATE PARTITION OF vs ATTACH PARTITION

### CREATE PARTITION OF

Creates a brand-new partition and immediately attaches it to the parent table.

```sql
CREATE TABLE emp_4
PARTITION OF emp
FOR VALUES IN (7);
```

Use this when the partition does not already exist.

---

### ATTACH PARTITION

Attaches an existing standalone table as a partition.

```sql
ALTER TABLE emp
ATTACH PARTITION emp_4
FOR VALUES IN (7);
```

Use this when:

- Migrating existing data
- Bulk-loading data into a standalone table
- Converting legacy tables into partitions
- Minimizing downtime during migrations

---

## 2.9 DETACH PARTITION

A partition can be removed from the partition hierarchy without deleting its data.

```sql
ALTER TABLE emp
DETACH PARTITION emp_2;
```

After detaching:

- `emp_2` becomes a normal standalone table.
- Existing data remains unchanged.
- Queries against `emp` no longer include rows from `emp_2`.

This feature is commonly used for archiving historical data, exporting partitions, or migrating data to another system.

---

## 2.10 Useful Health Checks

Display the complete partition hierarchy:

```sql
SELECT *
FROM pg_partition_tree('emp');
```

List parent-child relationships:

```sql
SELECT
    inhparent::regclass AS parent_table,
    inhrelid::regclass AS partition_table
FROM pg_inherits;
```

Verify where each row is physically stored:

```sql
SELECT
    tableoid::regclass AS partition_name,
    *
FROM emp;
```

This is one of the most useful troubleshooting techniques when working with partitioned tables.

---

## Trainer Tips

- A `DEFAULT` partition acts as a safety net for unexpected partition key values.
- Monitor the `DEFAULT` partition regularly.
- A growing `DEFAULT` partition usually indicates that future partitions are missing.
- Use a temporary `CHECK` constraint before `ATTACH PARTITION` to avoid expensive validation scans.
- Use `tableoid::regclass` whenever you need to determine which partition actually stores a row.

---

## Common Mistakes

- Not creating a `DEFAULT` partition when unexpected values are possible.
- Leaving rows in the `DEFAULT` partition for long periods.
- Attempting to `ATTACH PARTITION` while matching rows still exist in the `DEFAULT` partition.
- Forgetting to move rows from the `DEFAULT` partition before attaching a new partition.
- Forgetting to use `tableoid::regclass` while troubleshooting partition placement.
---

# Hands-on Exercises

1. Add emp_5 for values (10,11).
2. Create DEFAULT.
3. Insert random IDs.
4. Use tableoid to verify routing.
5. DETACH emp_3 and query it independently.

---

# Interview Questions

1. When should LIST partitioning be used?
2. What happens without DEFAULT?
3. Why does ATTACH fail when DEFAULT already contains matching rows?
4. Difference between ATTACH and CREATE PARTITION OF?
5. What does tableoid show?

---

# Summary

You now understand LIST partitioning, DEFAULT partitions, routing, ATTACH/DETACH and production considerations.

Next Chapter: RANGE Partitioning.
