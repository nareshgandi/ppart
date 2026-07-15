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

# 2.7 Creating a New LIST Partition Later

Suppose emp_default already contains:

```text
32432
```

Attempting

```sql
ALTER TABLE emp
ATTACH PARTITION emp_4
FOR VALUES IN (32432);
```

fails because that value already exists in DEFAULT.

Correct workflow:

1. Create standalone table.
2. Move matching rows.
3. Delete from DEFAULT.
4. ATTACH PARTITION.

---

# 2.8 ATTACH vs CREATE PARTITION OF

CREATE PARTITION OF
- Creates and attaches in one step.

ATTACH PARTITION
- Attaches an existing table.
- Common during migrations.

---

# 2.9 DETACH Partition

```sql
ALTER TABLE emp
DETACH PARTITION emp_2;
```

The table still exists but is no longer part of the partition hierarchy.

---

# 2.10 Health Checks

```sql
SELECT * FROM pg_partition_tree('emp');

SELECT inhparent::regclass,
       inhrelid::regclass
FROM pg_inherits;
```

---

# Trainer Tips

• DEFAULT partitions are excellent safety nets.
• Monitor DEFAULT regularly.
• Empty DEFAULT partitions indicate good partition management.

---

# Common Mistakes

- Forgetting DEFAULT.
- Overusing DEFAULT.
- Trying to ATTACH a partition whose values already exist in DEFAULT.
- Forgetting tableoid during troubleshooting.

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
