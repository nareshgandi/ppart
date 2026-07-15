# PostgreSQL Advanced Mastery
## Volume 5 – Partitioning at Production Scale

# Chapter 1 – Introduction to PostgreSQL Partitioning

## Learning Objectives

After this chapter you will be able to:

- Explain why partitioning exists.
- Understand parent and child partitions.
- Differentiate partitioning from sharding.
- Explain partition pruning.
- Choose between LIST, RANGE and HASH partitioning.

---

# 1.1 Why Partitioning Exists

As databases grow, tables can reach hundreds of millions or even billions of rows.

Common challenges:

- Large indexes
- Longer VACUUM operations
- Slower backups
- Expensive DELETE operations
- Longer maintenance windows

Partitioning divides one logical table into multiple physical tables while keeping the application interface unchanged.

---

# 1.2 Partitioning vs Sharding

| Partitioning | Sharding |
|--------------|----------|
| One PostgreSQL cluster | Multiple PostgreSQL clusters |
| Transparent to application | Usually application aware |
| Managed by PostgreSQL | Managed by application/infrastructure |

---

# 1.3 PostgreSQL Partition Architecture

```text
                Application
                     |
             INSERT / SELECT
                     |
            Parent Table (Virtual)
                     |
      +--------------+--------------+
      |              |              |
  Partition A    Partition B    Partition C
```

The parent table stores metadata and routing information. User rows are stored only in child partitions.

---

# 1.4 Parent Table

The parent table:

- Defines the partition key
- Defines the partitioning strategy
- Normally stores no rows
- Is the object applications query

---

# 1.5 Child Partitions

Each child partition is an independent PostgreSQL table with:

- Heap
- Indexes
- Statistics
- Visibility Map
- Free Space Map

---

# 1.6 INSERT Routing

```text
INSERT
   |
Partition Router
   |
LIST / RANGE / HASH
   |
Correct Child Partition
```

---

# 1.7 SELECT Processing

```text
SELECT
   |
Planner
   |
Partition Pruning
   |
Only matching partitions are scanned
```

Partition pruning is one of the main reasons partitioning can improve query performance.

---

# 1.8 Types of Declarative Partitioning

### LIST
Best for discrete values like region or department.

### RANGE
Best for dates, timestamps, IDs, or salary ranges.

### HASH
Best when even distribution is required.

---

# 1.9 Benefits

- Faster maintenance
- Faster archival
- Partition pruning
- Smaller indexes
- Independent VACUUM/ANALYZE

---

# 1.10 When NOT to Partition

Avoid partitioning when:

- Tables are small.
- There is no suitable partition key.
- Workloads don't benefit from pruning.

---

# 1.11 Useful Catalogs

```sql
SELECT * FROM pg_partitioned_table;

SELECT * FROM pg_partition_tree('parent_table');

SELECT * FROM pg_inherits;
```

---

# Trainer Tips

- Partitioning is primarily a scalability and maintenance feature.
- Good partition keys matter more than the number of partitions.
- Partitioning does not replace indexing.

---

# Interview Questions

1. What is a partitioned table?
2. What is a parent table?
3. Does the parent table store rows?
4. What is partition pruning?
5. LIST vs RANGE vs HASH?

---

# Summary

This chapter introduced the motivation, architecture and terminology behind PostgreSQL declarative partitioning.

The next chapter covers LIST partitioning with complete hands-on labs.
