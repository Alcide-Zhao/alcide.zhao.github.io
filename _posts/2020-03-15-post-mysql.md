---
title: 'Database and MySQL'
date: 2020-03-15
permalink: /posts/2020/03/post-MySQL-db/
tags:
  - database
  - SQL
  - DBMS
  - RDBMS
---

SQL (structured querry language) is a standard language for storing, manipulating and retrieving data in databases.

**Some quick reference pages**
[SQL keywords](https://www.w3schools.com/sql/sql_ref_keywords.asp)&emsp;[String, numeric and date function](https://www.w3schools.com/sql/sql_ref_mysql.asp)&emsp;[server function](https://www.w3schools.com/sql/sql_ref_sqlserver.asps)&emsp;[Operators](https://www.w3schools.com/sql/sql_operators.asp)&emsp;Data types](https://www.w3schools.com/sql/sql_datatypes.asp)

## Database 
- List existing databases: `show databases`
- Create database: `create database [db_name]`
- Delete database: `drop [db_name]`
- use database: `use [db_name]`
- show which database is using: `select databases();`
- backup database: `BACKUP DATABASE [db_name] TO DISK = 'filepath'; `. A differential back up only backs up the parts of the database that have changed since the last full database backup: `BACKUP DATABASE databasename TO DISK = 'filepath' WITH DIFFERENTIAL;`

## Table manipulation
- `create` table statement: `create table [tb_name] (*argv);`
<pre>
CREATE TABLE tb_name (
    column1 datatype,
    column2 datatype,
    column3 datatype,
   ....
); 
</pre>
  1. `NOT NULL ` to enforces a column to NOT accept NULL values;
  2. `NIQUE` constraint ensures that all values in a column are different.
  3. `PRIMARY KEY` uniquely identifies each record in a table.
  4. `FOREIGN KEY`  to link two tables together: `FOREIGN KEY (PersonID) REFERENCES Persons(PersonID)`
  5. `CHECK` constraint is used to limit the value range that can be placed in a column: ` CHECK (Age>=18)`
  6. `DEFAULT`to provide a default value for a column
  7. `Auto-increment` allows a unique number to be generated automatically when a new record is inserted into a table.

- `DROP` table statement: `drop table [tb_name]`
- `INSERT INTO` table statement: `INSERT INTO Customers (CustomerName, City, Country)
VALUES ('Cardinal', 'Stavanger', 'Norway');`
- `UPDATE` table statement: `UPDATE [tb_name]
SET [column1] = [value1, column2 = value2, ...]
WHERE [condition]; `
- `delete from ` table statement: `DELETE FROM [tb_name] WHERE [condition];`
- To add a column in a table: `ALTER TABLE table_name ADD column_name datatype; `; To delete a column in a table: `ALTER TABLE table_name DROP COLUMN column_name; `

	

## Table query

- Query tables: `select * from [tb_name];`

- Select `distinct` statement: `select distinct ([column_name]) from [tb_name];`

- `Count()` no. of items: `SELECT COUNT(DISTINCT Country) FROM Customers;`; `avg()'` to average over the selected; `sum()` to return the sum of the selected; `MIN()` and `MAX()` to return the the smallest/largest value of the sleceted; 

- `Where` to filter records: `SELECT * FROM Customers WHERE Country='Mexico'; `. The following operators can be used in the WHERE clause: `=, !=, >, <, >=, <=, like, not like, between... and..., in, not in, and, or, not, is null, is not null`

- `order by` to sort results in `ascending(asc)` or `desceding(desc)` order.  Can be ordered by several column: `SELECT * FROM Customers ORDER BY Country ASC, CustomerName DESC;`

- `limit` results: `SELECT * FROM Customers LIMIT 3; `

- `LIKE` to match wildcard, where `_` for single character, `%` for any number of characters,  `[charlist]`  for anyone inside [], and `[!character]` for anyone not inside [].

-  Alisa for column syntax `as`: `SELECT [column_name] AS alias_name
FROM [tb_name];`

- Alias for table syntax: `SELECT o.OrderID, o.OrderDate, c.CustomerName
FROM Customers AS c, Orders AS o
WHERE c.CustomerName="Around the Horn" AND c.CustomerID=o.CustomerID;`

- A JOIN clause is used to combine rows from two or more tables, based on a related column between them. 
<pre>
SELECT Orders.OrderID, Customers.CustomerName, Shippers.ShipperName
FROM ((Orders
INNER JOIN Customers ON Orders.CustomerID = Customers.CustomerID)
INNER JOIN Shippers ON Orders.ShipperID = Shippers.ShipperID); 
</pre>

- The LEFT JOIN keyword returns all records from the left table (table1), and the matched records from the right table (table2). The result is NULL from the right side, if there is no match.
<pre>
SELECT [column_name(s)]
FROM [table1]
LEFT JOIN [table2]
ON [table1.column_name] = [table2.column_name];
</pre>

- The RIGHT JOIN keyword returns all records from the right table (table2), and the matched records from the left table (table1). The result is NULL from the left side, when there is no match.
<pre>
SELECT [column_name(s)]
FROM [table1]
RIGHT JOIN [table2]
ON [table1.column_name] = [table2.column_name];
</pre>

- The UNION operator is used to combine the result-set of two or more SELECT statements: Each SELECT statement within UNION must have the same number of columns; The columns must also have similar data types; The columns in each SELECT statement must also be in the same order. 
<pre>
SELECT column_name(s) FROM table1
UNION
SELECT column_name(s) FROM table2; 
</pre>

- `GROUP BY` statement to groups rows that have the same values into summary rows. The GROUP BY statement is often used with aggregate functions (`COUNT, MAX, MIN, SUM, AVG`) to group the result-set by one or more columns.

- `HAVING` statement was added to SQL because the WHERE keyword could not be used with aggregate functions.
<pre>
SELECT [column_name(s)]
FROM [table_name]
WHERE [condition]
GROUP BY [column_name(s)]
HAVING [condition]
ORDER BY [column_name(s)];
</pre>

- The `EXISTS` operator is used to test for the existence of any record in a subquery.
<pre>
SELECT column_name(s)
FROM table_name
WHERE EXISTS
(SELECT column_name FROM table_name WHERE condition); 
</pre>
- The `ANY` and `ALL` operators are used with a `WHERE` or `HAVING` clause.

- The `SELECT INTO` statement copies data from one table into a new table [in the other database], you can change the column names in the new table using the `as` operator
<pre>
SELECT column1, column2, column3, ...
INTO newtable [IN externaldb] AS c1,c2,c3,...
FROM oldtable
WHERE condition; 
</pre>

- The `CASE` statement goes through conditions and returns a value when the first condition is met (like an IF-THEN-ELSE statement).
<pre>
SELECT OrderID, Quantity,
CASE
    WHEN Quantity > 30 THEN "The quantity is greater than 30"
    WHEN Quantity = 30 THEN "The quantity is 30"
    ELSE "The quantity is under 30"
END AS QuantityText
FROM OrderDetails; 
</pre>

## Stored procedure

- A stored procedure is a prepared SQL code that you can save, so the code can be reused over and over again.

<pre>
-- Stored Procedure Syntax
CREATE PROCEDURE procedure_name
AS
sql_statement
GO; 

-- Execute a Stored Procedure
EXEC procedure_name; 
</pre>

- Stored Procedure With Parameters
<pre>
-- Tcreates a stored procedure that selects Customers from a particular City with a particular PostalCode from the "Customers" table 
CREATE PROCEDURE SelectAllCustomers @City varchar(30), @PostalCode varchar(10)
AS
SELECT * FROM Customers WHERE City = @City AND PostalCode = @PostalCode
GO;

-- Execute the stored procedure above as follows:
EXEC SelectAllCustomers @City = "London", @PostalCode = "WA1 1DP"; 
</pre>

## Comments
The comments may depend on which SQL drive you use.  
- single Line Comments with `--`
- Block comments start with `/*` and end with `*/`.

## SQL Dates

- DATE - format YYYY-MM-DD
- DATETIME - format: YYYY-MM-DD HH:MI:SS
- TIMESTAMP - format: YYYY-MM-DD HH:MI:SS
- YEAR - format YYYY or YY
- NOW() - format: YYYY-MM-DD HH:MI:SS










