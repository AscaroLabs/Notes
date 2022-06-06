# Chapter 2. The SQL Language

---

__Table of Contents__

1. [Introduction](#introduction)
2. [Concepts](#concepts)
3. [Creating a New Table](#creating-a-new-table)
4. [Populating a Table With Rows](#populating-a-table-with-rows)
5. [Querying a Table](#querying-a-table)
6. [Joins Between Tables](#joins-between-tables)
7. [Aggregate Functions](#aggregate-functions)
8. [Updates](#updates)
9. [Deletions](#deletions)

## Introduction

This chapter provides an overview of how to use SQL to perform simple operations. This tutorial is only intended to give you an introduction and is in no way a complete tutorial on SQL. 

Examples in this manual can also be found in the PostgreSQL source distribution in the directory `src/tutorial/`.

```sh
$ psql -s mydb

...

mydb=> \i basics.sql
```

## Concepts

PostgreSQL is a _relational database management system_ (RDBMS). That means it is a system for managing data stored in _relations_. Relation is essentially a mathematical term for _table_.

Each table is a named collection of _rows_. Each row of a given table has the same set of named _columns_, and each column is of a specific data type. 

Tables are grouped into databases, and a collection of databases managed by a single PostgreSQL server instance constitutes a database _cluster_.

## Creating a New Table

You can create a new table by specifying the table name, along with all column names and their types:

```sql
CREATE TABLE weather (
    city            varchar(80),
    temp_lo         int,           -- low temperature
    temp_hi         int,           -- high temperature
    prcp            real,          -- precipitation
    date            date
);
```

```sql
CREATE TABLE cities (
    name            varchar(80),
    location        point
);
```

The point type is an example of a PostgreSQL-specific data type.

Finally, it should be mentioned that if you don't need a table any longer or want to recreate it differently you can remove it using the following command:

```sql
DROP TABLE tablename;
```

## Populating a Table With Rows

The `INSERT` statement is used to populate a table with rows:

```sql
INSERT INTO weather VALUES ('San Francisco', 46, 50, 0.25, '1994-11-27');
```

```sql
INSERT INTO cities VALUES ('San Francisco', '(-194.0, 53.0)');
```

The syntax used so far requires you to remember the order of the columns. An alternative syntax allows you to list the columns explicitly:

```sql
INSERT INTO weather (city, temp_lo, temp_hi, prcp, date)
    VALUES ('San Francisco', 43, 57, 0.0, '1994-11-29');
```

You could also have used COPY to load large amounts of data from flat-text files. This is usually faster because the COPY command is optimized for this application while allowing less flexibility than INSERT. An example would be:

```sql
COPY weather FROM '/home/user/weather.txt';
```

## Querying a Table

To retrieve data from a table, the table is _queried_. An SQL `SELECT` statement is used to do this.

```sql
SELECT * FROM weather;
```

```
   city      | temp_lo | temp_hi | prcp |    date
---------------+---------+---------+------+------------
 San Francisco |      46 |      50 | 0.25 | 1994-11-27
 San Francisco |      43 |      57 |    0 | 1994-11-29
 Hayward       |      37 |      54 |      | 1994-11-29
(3 rows)
```

You can write expressions, not just simple column references, in the select list. For example, you can do:

```sql
SELECT city, (temp_hi+temp_lo)/2 AS temp_avg, date FROM weather;
```

```
     city      | temp_avg |    date
---------------+----------+------------
 San Francisco |       48 | 1994-11-27
 San Francisco |       50 | 1994-11-29
 Hayward       |       45 | 1994-11-29
(3 rows)
```

A query can be “qualified” by adding a `WHERE` clause that specifies which rows are wanted. The WHERE clause contains a Boolean (truth value) expression, and only rows for which the Boolean expression is true are returned. The usual Boolean operators (AND, OR, and NOT) are allowed in the qualification. For example, the following retrieves the weather of San Francisco on rainy days:

```sql
SELECT * FROM weather
    WHERE city = 'San Francisco' AND prcp > 0.0;
```

```
     city      | temp_lo | temp_hi | prcp |    date
---------------+---------+---------+------+------------
 San Francisco |      46 |      50 | 0.25 | 1994-11-27
(1 row)
```

You can request that the results of a query be returned in sorted order:

```sql
SELECT * FROM weather
    ORDER BY city;
```

```
     city      | temp_lo | temp_hi | prcp |    date
---------------+---------+---------+------+------------
 Hayward       |      37 |      54 |      | 1994-11-29
 San Francisco |      43 |      57 |    0 | 1994-11-29
 San Francisco |      46 |      50 | 0.25 | 1994-11-27
```

You can request that duplicate rows be removed from the result of a query:

```sql
SELECT DISTINCT city
    FROM weather;
```

```
     city
---------------
 Hayward
 San Francisco
(2 rows)
```

## Joins Between Tables

Thus far, our queries have only accessed one table at a time. Queries can access multiple tables at once, or access the same table in such a way that multiple rows of the table are being processed at the same time. Queries that access multiple tables (or multiple instances of the same table) at one time are called _join_ queries.

```sql
SELECT * FROM weather JOIN cities ON city = name;
```

```
     city      | temp_lo | temp_hi | prcp |    date    |     name      | location
---------------+---------+---------+------+------------+---------------+-----------
 San Francisco |      46 |      50 | 0.25 | 1994-11-27 | San Francisco | (-194,53)
 San Francisco |      43 |      57 |    0 | 1994-11-29 | San Francisco | (-194,53)
(2 rows)
```

## Aggregate Functions

Like most other relational database products, PostgreSQL supports _aggregate functions_. An aggregate function computes a single result from multiple input rows.

```sql
SELECT max(temp_lo) FROM weather;
```

```
 max
-----
  46
(1 row)
```
```sql
SELECT city FROM weather
    WHERE temp_lo = (SELECT max(temp_lo) FROM weather);
```

```
     city
---------------
 San Francisco
(1 row)
```

Aggregates are also very useful in combination with `GROUP BY` clauses. For example, we can get the maximum low temperature observed in each city with:

```sql
SELECT city, max(temp_lo)
    FROM weather
    GROUP BY city;
```

```
     city      | max
---------------+-----
 Hayward       |  37
 San Francisco |  46
(2 rows)
```

which gives us one output row per city. Each aggregate result is computed over the table rows matching that city. We can filter these grouped rows using `HAVING`:

```sql
SELECT city, max(temp_lo)
    FROM weather
    GROUP BY city
    HAVING max(temp_lo) < 40;
```

```
  city   | max
---------+-----
 Hayward |  37
(1 row)
```

Finally, if we only care about cities whose names begin with “S”, we might do:

```sql
SELECT city, max(temp_lo)
    FROM weather
    WHERE city LIKE 'S%'
    GROUP BY city
    HAVING max(temp_lo) < 40;
```

It is important to understand the interaction between aggregates and SQL's `WHERE` and `HAVING` clauses. The fundamental difference between `WHERE` and `HAVING` is this: `WHERE` selects input rows before groups and aggregates are computed (thus, it controls which rows go into the aggregate computation), whereas `HAVING` selects group rows after groups and aggregates are computed. 

## Updates

You can update existing rows using the `UPDATE` command. 

```sql
UPDATE weather
    SET temp_hi = temp_hi - 2,  temp_lo = temp_lo - 2
    WHERE date > '1994-11-28';
```

## Deletions

Rows can be removed from a table using the `DELETE` command. 

```sql
DELETE FROM weather WHERE city = 'Hayward';
```

One should be wary of statements of the form

```sql
DELETE FROM tablename;
```

Without a qualification, `DELETE` will remove all rows from the given table, leaving it empty. The system will not request confirmation before doing this!