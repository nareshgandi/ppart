# PostgreSQL Demo: PL/pgSQL vs PL/Python using a CSV File

## Goal

Compare a typical CSV processing workflow using native PostgreSQL and
then PL/Python.

## Sample CSV

Save as `/home/postgres/employees.csv`

``` csv
emp_id,name,department,salary
1,Alice,IT,10000
2,Bob,HR,15000
3,Charlie,Finance,20000
4,David,IT,18000
5,Eve,HR,17000
```

## Create Table

``` sql
CREATE TABLE employees(
    emp_id INT,
    name TEXT,
    department TEXT,
    salary INT
);
```

## Load CSV using PostgreSQL

``` sql
\copy employees
FROM '/home/postgres/employees.csv'
DELIMITER ','
CSV HEADER;
```

## SQL / PLpgSQL Examples

### Total Salary

``` sql
SELECT SUM(salary) FROM employees;
```

### Average Salary

``` sql
SELECT AVG(salary) FROM employees;
```

### Highest Salary

``` sql
SELECT *
FROM employees
ORDER BY salary DESC
LIMIT 1;
```

### IT Employees

``` sql
SELECT *
FROM employees
WHERE department='IT';
```

### Average Salary per Department

``` sql
SELECT department,AVG(salary)
FROM employees
GROUP BY department;
```

### Add Bonus

``` sql
ALTER TABLE employees ADD COLUMN bonus NUMERIC;

UPDATE employees
SET bonus=salary*0.10;
```

# PL/Python

Install

``` sql
CREATE EXTENSION plpython3u;
```

## Load CSV

``` sql
CREATE OR REPLACE FUNCTION load_employees_csv(path text)
RETURNS text
AS $$
import csv

with open(path) as f:
    reader = csv.DictReader(f)

    plan = plpy.prepare(
        "INSERT INTO employees VALUES($1,$2,$3,$4)",
        ["int","text","text","int"]
    )

    for row in reader:
        plpy.execute(plan,[
            int(row["emp_id"]),
            row["name"],
            row["department"],
            int(row["salary"])
        ])

return "Loaded"
$$ LANGUAGE plpython3u;
```

Run

``` sql
TRUNCATE employees;
SELECT load_employees_csv('/home/postgres/employees.csv');
```

## Total Salary

``` sql
CREATE FUNCTION total_salary()
RETURNS integer
AS $$
rows=plpy.execute("SELECT salary FROM employees")
return sum(r["salary"] for r in rows)
$$ LANGUAGE plpython3u;
```

## Average Salary

``` sql
CREATE FUNCTION average_salary()
RETURNS numeric
AS $$
rows=plpy.execute("SELECT salary FROM employees")
values=[r["salary"] for r in rows]
return sum(values)/len(values)
$$ LANGUAGE plpython3u;
```

## Highest Salary

``` sql
CREATE FUNCTION highest_salary()
RETURNS text
AS $$
r=plpy.execute("SELECT name FROM employees ORDER BY salary DESC LIMIT 1")
return r[0]["name"]
$$ LANGUAGE plpython3u;
```

## IT Employees

``` sql
CREATE FUNCTION it_employees()
RETURNS text[]
AS $$
rows=plpy.execute("SELECT name FROM employees WHERE department='IT'")
return [r["name"] for r in rows]
$$ LANGUAGE plpython3u;
```

## Add Bonus

``` sql
CREATE FUNCTION add_bonus()
RETURNS text
AS $$
plpy.execute("UPDATE employees SET bonus=salary*0.10")
return "Done"
$$ LANGUAGE plpython3u;
```

# Comparison

  Task       SQL/PLpgSQL     PL/Python
  ---------- --------------- ----------------
  CSV Load   `\copy`{=tex}   csv module
  Sum        SUM()           Python sum()
  Average    AVG()           Python list
  Highest    ORDER BY        Python + SQL
  Filter     WHERE           Python or SQL
  Bonus      UPDATE          plpy.execute()

# Recommendation

Use SQL for filtering, joins, aggregations and bulk operations.

Use PL/Python when Python libraries or file processing significantly
simplify the solution. It complements PostgreSQL rather than replacing
SQL.
