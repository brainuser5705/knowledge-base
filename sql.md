# Structured Query Language

*Query, manipulate, transform data from relational database*

- A relational database is a collection of related tables. 
  - Each table has a fixed number of named columns (attributes/properties of the table) and any number of rows.
- Table is an entity (ex. dog)
  - Each row is an instance of that entity (ex. Pug, Collie)
  - Each column represents shared properties of those instances (ex. Color of fur, length of tail)
- SQL queries and columns don’t have to be capitalized, but it’s convention to.

### MAKING QUERIES

#### `SELECT` *(queries statements)*

- Specific columns

  ```sql
  SELECT column, another_column …
  FROM mytable;
  ```

- All columns

  ```sql
  SELECT *
  FROM mytable;
  ```

#### `WHERE` clause *(Each constraint is applied to each row)*

- Syntax

  ```sql 
  SELECT *
  FROM mytable
  WHERE condition
    AND/OR another_condition
    AND/OR ....;
  ```

- Operators

- - Comparison operators (!=, ==, <, >, etc.)
    `col_name != 4`
  - Number is within range (inclusive)
    `col_name BETWEEN 1.5 AND 10.5`
  - Number is not within range (inclusive)
    `col_name NOT BETWEEN 1 AND 10`
  - Number is in list
    `col_name IN (2,4,6)`
  - Number is not in list
    `col_name NOT IN (1,3,5)`

- String specified operators

- - <u>_Note_</u>: strings must be in quotes to distinguish between SQL keywords

  - Case sensitive equality (exact match)
    `col_name = “abc”`

  - Case sensitive inequality
    `col_name != “abc” or col_name <> “abc”`

  - Case insensitive equality
    `col_name LIKE “abc”`

  - Case insensitive inequality
    `col_name NOT LIKE “abc”`

  - - Used with LIKE and NOT LIKE

    - - `% `: place in string to match a sequence of 0 or more characters
        `col_name LIKE “%AT%”`
        This will match with “AT”, “ATTIC”, “CAT”, or “BATS”
      - `_` : place in string to match a single character
        `col_name LIKE “AN_”`
        This will match with “AND” but not “AN”

  - String exist in list
    `col_name IN (“A”,”B”,”C”)`

  - String does not exist in list
    `col_name NOT IN (“D”, “E”, “F”)`

#### `DISTINCT` *(remove rows that have a duplicate column value(s))*

- ```sql 
  SELECT DISTINCT column, another_column, …
  FROM myTable
  WHERE condition(s);
  ```

- `DISTINCT` column must be first before any other column.

#### `ORDER BY` *(Ordering the results based on specified column)*

- ```sql
  SELECT column, another_column, …
  FROM myTable
  WHERE condition(s)
  ORDER BY column ASC/DESC;
  ```

- `ASC` is default.

- Can order by multiple columns with their own specified order: the leftmost ones have a higher precedence than the rightmost

#### `LIMIT and OFFSET` *(limit the number of rows to return and by an optional offset to start from)*

- ```sql
  SELECT column, another_column, …
  FROM myTable
  WHERE condition(s)
  ORDER BY column ASC/DESC
  LIMIT num_limit OFFSET num_offset;
  ```

- If the `OFFSET` is 2, then it will skip the first 2 rows and start at row 3.

#### **Expressions** 

- Each database system has their own supported functions for strings, math, date, etc.
  Ex. `ABS()`
- You can use expressions in your queries to save post processing time
- You can assign as alias using the `AS` keywords to expressions, columns, and tables

- ```sql 
  SELECT particle_speed / 2.0 AS half_particle_speed
  FROM physics_data_table AS phys_data
  WHERE ABS(particle_position) * 10.0 > 500;
  ```

#### **Aggregate Expressions**

- ```sql
  SELECT AGG_FUNC(column or expression) AS aggregate_description
  FROM table;
  ```

- **Common Aggregate Functions**

  - Number of rows in the group if no column name is specified, else the number of non-NULL values if specified
    `COUNT (*)`, `COUNT(column)`
  - Smallest numerical value in specified column
    `MIN(column)`
  - Largest numerical value in specified column
    `MAX(column)`
  - Average numerical value in specified column
    `AVG(column)`
  - Sum of all numerical values in specified column
    `SUM(column)`

  

#### `GROUP BY` *(groups rows that have the same value in the column specified)*

- Apply the aggregate function to individual groups of data of the group, creating as many results as there are distinct values in the group
  `GROUP BY column;`

- `GROUP BY` is placed after the WHERE clause (which filters the rows that are to be grouped). 

#### `HAVING` *(Filter the grouped rows)*

- ```sql 
  SELECT group_by_column, AGG_FUNC(expression) AS aggregate_description
  FROM table
  WHERE condition
  GROUP BY column
  HAVING group_conditon;
  ```

- Can only work with columns that appear in the `GROUP BY` clause or used in an aggregate function



### WORKING WITH MULTIPLE TABLES

- Database normalization is when data is spread across multiple tables.
  - It allows data to grow independently of each other (ex. Dog food and the type of dogs). 
  - Trade off is that queries will have to find data from many tables within the database.
  - We need to write a query that can combine all the data we need about an entity from different tables.
- A primary key is used to identify an entity uniquely across the database.
  - The most common type is an auto-incrementing integer (space-efficient).

#### `(INNER) JOIN` and `ON` *(join row data across two tables on a primary key)*

- ```sql
  SELECT column, another_column…
  FROM myTable
  INNER JOIN another_table
    ON myTable.id = another_table.id
  WHERE condition(s)
  ORDER BY column, … ASC/DESC
  LIMIT num_limit OFFSET num_offset;
  ```

- It matches rows from the first table and the second table that have the same key.

- The result row will be a table with combined columns from both tables.

- If there are no matching column names, optional dot operator for columns in the conditions.

- The `SELECT` query can contain columns from both tables.

#### Outer `JOINS` *(for asymmetric data when matching is not 1-to-1)*

1. `LEFT (OUTER) JOIN`
   Includes rows from first table regardless of if match with second table
2. `RIGHT (OUTER) JOIN`
   Includes rows from second table regardless of if match with first table
3. `FULL (OUTER) JOIN`
   Includes rows from both tables regardless of if there is match

#### `NULL` values

- Good to reduce the possibility of a `NULL` value because of maintenance

- Alternative is a data-type appropriate default value, but sometimes `NULL` is more appropriate

- `IS/IS NOT NULL`

  ```sql 
  SELECT column, another_column, …
  FROM mytable
  WHERE column IS/IS NOT NULL;
  ```



### Complete SELECT query

```sql
SELECT DISTINCT column, AGG_FUNC(column_or_expression), …
FROM mytable
  JOIN another_table
    ON mytable.column = another_table.column
  WHERE constraint_expression
  GROUP BY column
  HAVING constraint_expression
  ORDER BY column ASC/DESC
  LIMIT count OFFSET count;
```

**Order of Execution**

1. `FROM` and `JOIN`

2. 1. determine the total working set of data

3. `WHERE`

4. 1. rows that do not satisfy constraint are discarded
   2. Aliases in select part are not accessible in most databases*
   3. Can only access columns directly from tables in the FROM clause*

  (* works on SQLBolt, but maybe not mySQL?)

1. `GROUP BY`

2. 1. Remaining rows are grouped (only needed in aggregate functions)

3. `HAVING`

4. 1. Rows that do not satisfy constraint are discarded
   2. Aliases cannot be referenced yet.

5. `SELECT`

6. 1. Expressions in SELECT are computed

7. `DISTINCT`

8. 1. Remaining rows with duplicate values in DISTINCT columns are discarded.

9. `ORDER BY`

10. 1. Order the results by specified order
    2. Since expressions in SELECT have been computed, aliases can be referenced now.

11. `LIMIT and OFFSET`

12. 1. Rows that fall out of range are discarded.

### MAKING TABLES

A database schema describes:

- structure of each table
- the datatypes of each column

#### `INSERT` *(insert new data into table)*

- Insert with values for all columns

  ```sql
  INSERT INTO table
  VALUES (value_or_expr, another_value_or_expr, …),
      (value_or_expr_2, another_value_or_expr2, …)
      …;
  ```

- Insert into specific columns (where skipped columns have default values)

  ```sql
  INSERT INTO table
  (column, another_column, …)
  VALUES (value_or_expr, another_value_or_expr, …),
      (value_or_expr_2, another_value_or_expr2, …)
      …;
  ```

- The number of values need to match the number of columns specified.

#### `UPDATE` *(update existing data)*

- UPDATE table

  ```sql
  SET column = value_or_expr,
      other_column = another_value_or_expr,
      …
  WHERE condition;
  ```

- You have to specify exactly which columns to update.

- `WHERE` clause is needed to specify rows, otherwise all rows will be updated

#### `DELETE` *(delete rows of the table)*

- ```sql
  DELETE FROM table
  WHERE condition;
  ```

- If `WHERE` clause is left out, then all rows will be removed.

#### `CREATE TABLE` *(creates a new table when you have new entities and relationship)*

- ```sql
  CREATE TABLE IF NOT EXISTS table (
    column_name DataType TableConstraint DEFAULT default_value,
    …
  );
  ```

- The structure is defined by its table schema or the series of columns

- TableConstraint and DEFAULT are optional

- If a table already exists with the same name, SQL will throw an error. `IF NOT EXISTS` clause will suppress the error and skip the table creation.

- **Common Data types**

- - `INTEGER`, `BOOLEAN` (0 or 1 in some systems)

  - `FLOAT`, `DOUBLE`, `REAL`

  - `CHARACTER(num_chars)`, `VARCHAR(num_chars)`, `TEXT`

  - - The distinction is generally the database efficiency in handling these types.

  - `DATE`, `DATETIME`

  - `BLOB` (binary data, for files)

- Common Table constraints
   These limit what values can be inserted into the column
  - `PRIMARY KEY` (values in each row are unique and used to identify the rwo)
  - `AUTO_INCREMENT` (for integer values)
  - `UNIQUE` (does not have to be the primary key)
  - `NOT NULL` (NULL value cannot be inserted)
  - `CHECK(expression)` (used to test whether the value is valid, ex: if the value is positive)
  - `FOREIGN KEY` (Consistency check to ensure each value in this column corresponds to another value in a column in another table)

#### `ALTER TABLE` *(add, remove, modify columns and table constraints)*

- Adding columns

  ```sql
  ALTER TABLE table
  ADD column DataType OptionalTableConstraint DEFAULT default_value;
  ```

- - in some databases you can specify where to add the column

- Removing columns

  ```sql
  ALTER TABLE table
  DROP column_to_be_deleted;
  ```

  - some databases don’t support this, you would have to create and migrate

- Renaming table

  ```sql
  ALTER TABLE table
  RENAME TO new_table_name;
  ```

#### `DROP TABLE IF EXISTS` *(removes an entire table from database)*

- ```sql
  DROP TABLE IF EXISTS table;
  ```

- If another table is dependent on the columns in the table, you will have to update or delete all dependent tables



### Subqueries (nested queries)

- Can be referenced anywhere a normal table can be referenced

- - `JOIN` subqueries with other tables
  - Test against subqueries in `WHERE` and `HAVING` clauses
  - `SELECT` clause to return data directly from subquery

- Each subquery must be enclosed in parentheses

- Correlated subqueries where inner query is dependent on column or alias (when the tables from both queries are the same) from outer query

- - This example is where you are testing an employee’s revenue to their department’s average revenue. In the subquery, the `WHERE` clause makes sure that the average is generated from employees that are in the same department as the current employee that is being checked.

  - ```sql
    SELECT *
    FROM employees
    WHERE revenue > 
      (SELECT AVG(revenue)
       FROM employees AS dept_employees
           WHERE employees.department = dept_employees.department);
    ```

- Existence test (when constraints are based on current data)
  This test whether a value exists in a dynamic list of values.

  ```sql
  SELECT *, …
  FROM table
  WHERE column
    IN/NOT IN (SELECT another_column
          FROM another_table);
  ```

### Sets

When working with multiple tables, you can append the results of two queries if the results have the same column count, order, and datatype

SELECT column, another_column
  FROM mytable
UNION / UNION ALL / INTERSECT / EXCEPT

SELECT other_column
  FROM other_table
ORDER BY column DESC (Union is used before order and limit)
LIMIT n;



UNION: append without duplicate rows
UNION ALL: append all rows including duplicate
INTERSECT: only rows that are identical in both results table are returned
EXCEPT: only rows that exists in the first result but not the second result

(some database system have INTERSECT ALL and EXCEPT ALL)





PHP and MySQL (Prepared Statements)



1. SQL query template with unspecified parameters as ‘?’ or with PDO ‘:parameter_name’
   Ex. INSERT INTO table VALUES(?,?,?)
2. Database parses, compiles and performs query optimization with the template before any execution.
   *Query optimization is a feature where it attempts to find the most efficient way of executing a query by considering all possible query plans.*
3. To execute with a prepared statement, the application/code binds the values to the template and executes it. 



Why?

- Reduces parsing speed as the database performed query optimization
- You only need to send the parameters, not the entire query
- Useful against SQL injections as parameters are sent using a different protocol







Other



Numbers in replace of column names

Non equi-join and aliases for the same table

![img](https://lh6.googleusercontent.com/CBfLzWhzEHRnf9mRbJEpWabsoyIzz0yCRVAxE26QuB56Wmo45sOn0SLrT74huzr11Uu9qAvvy4ywQuYZ9oMNN3ISoscFMvON5WcihmHA9UnX-S-LU_VkiTxTsyoMiEUZaNG5u3JR)

You can give table aliases without the AS keyword (optional).



Can have multiple JOIN statements in one query (ex. Joining 3 tables)

Aggregate functions not allowed in WHERE clause



When using join, columns in the select column must either be in an aggregate function or a group by clause