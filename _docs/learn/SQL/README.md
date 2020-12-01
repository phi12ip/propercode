# SQL

## Joins
<!---
Source:
https://build.affinity.co/understanding-tricky-joins-and-multi-table-updates-in-postgresql-using-boolean-algebra-7b329606ca45
by: Paul Martinez
--->
JOIN types

We’ll first create two tables with some sample data and use them to give a quick rundown of the different types of joins.

``` sql
CREATE TABLE table_1 (id INTEGER, value1 TEXT);
CREATE TABLE table_2 (id INTEGER, value2 TEXT);INSERT INTO table_1 VALUES (1, 'a'), (2, 'b'), (3, 'c');
INSERT INTO table_2 VALUES (1, 'W'), (1, 'X'), (3, 'Y'), (5, 'Z');SELECT * FROM table_1;
--  id | value1
-- ----+--------
--   1 | a
--   2 | b
--   3 | cSELECT * FROM table_2;
--  id | value2
-- ----+--------
--   1 | W
--   1 | X
--   3 | Y
--   5 | Z
```

JOIN TYPE can be one of the following (words in square brackets are optional), each generating a different result set:

* [INNER] JOIN: For each row R1 in T1, the joined table has a row for every row R2 in T2 that satisfies the join condition:

``` sql
SELECT * FROM table_1 JOIN table_2 ON table_1.id = table_2.id;
--  id | value1 | id | value2
-- ----+--------+----+--------
--   1 | a      |  1 | W
--   1 | a      |  1 | X
--   3 | c      |  3 | Y
```

* LEFT [OUTER] JOIN: First an inner join is performed, then a row is added with NULL values for the columns of T2 for each row R1 in T1 that was not matched with a row in T2. This ensures that every row of T1 will be present in the outputted table:

``` sql
SELECT * FROM table_1 LEFT JOIN table_2 ON table_1.id = table_2.id;
--  id | value1 | id | value2
-- ----+--------+----+--------
--   1 | a      |  1 | W
--   1 | a      |  1 | X
--   2 | b      |    |
--   3 | c      |  3 | Y
```

* RIGHT [OUTER] JOIN: The same as a left outer join, but with the roles of T1 and T2 swapped, ensuring that each row of T2 will be present in the outputted table.

``` sql
SELECT * FROM table_1 RIGHT JOIN table_2 ON table_1.id = table_2.id;
--  id | value1 | id | value2
-- ----+--------+----+--------
--   1 | a      |  1 | W
--   1 | a      |  1 | X
--   3 | c      |  3 | Y
--     |        |  5 | Z
```

* FULL [OUTER] JOIN: The combination of a left outer join and a right outer join. An inner join is performed, then a new row is added for each unmatched row from T1 and T2:

``` sql
SELECT * FROM table_1 FULL JOIN table_2 ON table_1.id = table_2.id;
--  id | value1 | id | value2
-- ----+--------+----+--------
--   1 | a      |  1 | W
--   1 | a      |  1 | X
--   2 | b      |    |
--   3 | c      |  3 | Y
--     |        |  5 | Z
```

* CROSS JOIN: Every row in T1 with every row in T2, forming the Cartesian product. When using this join type there is no ON <expression> clause specifying a join condition.

``` sql
SELECT * FROM table_1 CROSS JOIN table_2;
--  id | value1 | id | value2
-- ----+--------+----+--------
--   1 | a      |  1 | W
--   1 | a      |  1 | X
--   1 | a      |  3 | Y
--   1 | a      |  5 | Z
--   2 | b      |  1 | W
--   2 | b      |  1 | X
--   2 | b      |  3 | Y
--   2 | b      |  5 | Z
--   3 | c      |  1 | W
--   3 | c      |  1 | X
--   3 | c      |  3 | Y
--   3 | c      |  5 | Z
```

There is another, less commonly used way to select data from multiple tables. Tables can simply be listed one after another in a comma separated list in the FROM clause, and this is identical to performing a cross join. (At least when only two tables are listed and no filtering happens in a WHERE clause.)

``` sql
SELECT * FROM table_1, table_2;
--  id | value1 | id | value2
-- ----+--------+----+--------
--   1 | a      |  1 | W
--   1 | a      |  1 | X
--   1 | a      |  3 | Y
--   1 | a      |  5 | Z
--   2 | b      |  1 | W
--   2 | b      |  1 | X
--   2 | b      |  3 | Y
--   2 | b      |  5 | Z
--   3 | c      |  1 | W
--   3 | c      |  1 | X
--   3 | c      |  3 | Y
--   3 | c      |  5 | Z
```

Notice that this is also the result one would get performing a regular join with TRUE as a condition. And going the other way, notice that one can perform an inner join on a condition C by performing a cross join and then using a WHERE clause with condition C. That is, the statements SELECT * FROM T1 INNER JOIN T2 ON C and SELECT * FROM T1 CROSS JOIN T2 WHERE C are equivalent.

