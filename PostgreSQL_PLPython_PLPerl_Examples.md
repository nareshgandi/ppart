# PostgreSQL PL/Python and PL/Perl Extensions

## Objective

Understand when PostgreSQL's procedural language extensions are useful
and when plain SQL/PLpgSQL is sufficient.

------------------------------------------------------------------------

# Do I Need These Extensions?

Most PostgreSQL users **do not**.

Use them when: - Complex string manipulation is easier in Python/Perl -
JSON transformation becomes cumbersome in SQL - Reusing small Python
logic close to the data is beneficial

Avoid them for: - Calling external APIs - Running AI/ML models - Heavy
computations - General application business logic

------------------------------------------------------------------------

# Install Extensions

``` sql
CREATE EXTENSION plpython3u;
CREATE EXTENSION plperl;
```

> `plpython3u` is an **untrusted** language. Only superusers can create
> PL/Python functions.

------------------------------------------------------------------------

# Example 1 -- Uppercase

## Without Extension

``` sql
SELECT upper('postgres');
```

## Using PL/Python

``` sql
CREATE OR REPLACE FUNCTION py_upper(text)
RETURNS text
AS $$
return args[0].upper()
$$ LANGUAGE plpython3u;

SELECT py_upper('postgres');
```

------------------------------------------------------------------------

# Example 2 -- Reverse Text

## Without Extension

``` sql
SELECT reverse('PostgreSQL');
```

(For older PostgreSQL versions without `reverse()`, this requires more
complex SQL.)

## Using PL/Perl

``` sql
CREATE OR REPLACE FUNCTION reverse_text(text)
RETURNS text
AS $$
return scalar reverse $_[0];
$$ LANGUAGE plperl;

SELECT reverse_text('PostgreSQL');
```

------------------------------------------------------------------------

# Example 3 -- SHA256 Hash

## Without Extension

Requires the pgcrypto extension.

``` sql
CREATE EXTENSION pgcrypto;

SELECT encode(
    digest('postgres','sha256'),
    'hex'
);
```

## Using PL/Python

``` sql
CREATE OR REPLACE FUNCTION sha256(text)
RETURNS text
AS $$
import hashlib
return hashlib.sha256(args[0].encode()).hexdigest()
$$ LANGUAGE plpython3u;

SELECT sha256('postgres');
```

------------------------------------------------------------------------

# Example 4 -- Count Words

## Without Extension

``` sql
SELECT
array_length(
    regexp_split_to_array(
        'PostgreSQL is an amazing database',
        '\s+'
    ),
1);
```

## Using PL/Python

``` sql
CREATE FUNCTION word_count(text)
RETURNS integer
AS $$
return len(args[0].split())
$$ LANGUAGE plpython3u;

SELECT word_count('PostgreSQL is an amazing database');
```

------------------------------------------------------------------------

# Example 5 -- JSON Processing

``` sql
CREATE TABLE orders
(
    id int,
    payload jsonb
);

INSERT INTO orders VALUES
(1,'{"customer":"Naresh","amount":1200}');
```

## Without Extension

``` sql
SELECT payload->>'customer'
FROM orders;
```

## Using PL/Python

``` sql
CREATE FUNCTION customer_name(jsonb)
RETURNS text
AS $$
return args[0]["customer"]
$$ LANGUAGE plpython3u;

SELECT customer_name(payload)
FROM orders;
```

------------------------------------------------------------------------

# Pros

-   Python syntax is concise
-   Excellent for string and JSON manipulation
-   Can reuse existing Python knowledge
-   Runs close to the data

# Cons

-   Requires superuser for PL/Python
-   Runs inside PostgreSQL backend processes
-   Poor choice for long-running work
-   Not suitable for AI inference or external service calls
-   Additional operational and security considerations

------------------------------------------------------------------------

# Recommendation

For most production systems:

-   SQL + PL/pgSQL for database logic
-   Application code for business logic
-   PL/Python only for small, focused helper functions
-   PL/Perl mainly for legacy environments or regex-heavy tasks
