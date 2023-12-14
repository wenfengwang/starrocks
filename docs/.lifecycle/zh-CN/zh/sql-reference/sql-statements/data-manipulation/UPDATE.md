---
displayed_sidebar: "Chinese"
---

# UPDATE

This statement is used to update a data row in a primary key model table.

From version 2.3 to 2.5, StarRocks provides the UPDATE statement, only supporting single-table UPDATE and not supporting common table expressions (CTE). Starting from version 3.0, StarRocks enriches the UPDATE syntax, supporting multi-table associations and CTE. If the table to be updated needs to be associated with other tables in the database, you can reference other tables in the FROM clause or CTE. Starting from version 3.1, partial updates of column mode are supported, suitable for scenarios involving a small number of columns but a large number of rows to accelerate the update speed.

## Instructions

For multi-table UPDATE, you need to ensure that the table expression in the FROM clause of the UPDATE statement can be converted into an equivalent JOIN query statement. Because when StarRocks actually executes the UPDATE statement, it will perform this conversion internally. Assuming the UPDATE statement is `UPDATE t0 SET v1=t1.v1 FROM t1 WHERE t0.pk = t1.pk;`, the table expression in the FROM clause can be converted to `t0 JOIN t1 ON t0.pk=t1.pk;`. Then, according to the result set of the JOIN query, StarRocks matches the data rows of the table to be updated and updates the values of the specified columns. If there are multiple rows of data in the result set that match a certain row of the table to be updated, the value of the specified column in this row of the table to be updated after the update is the value of the specified column of a random row in the result set.

## Syntax

**Single-table UPDATE**

If the data row of the table to be updated meets the WHERE condition, new values are assigned to the specified columns of this data row.

```SQL
[ WITH <with_query> [, ...] ]
UPDATE <table_name>
    SET <column_name> = <expression> [, ...]
    WHERE <where_condition>
```

**Multi-table UPDATE**

Based on the result set of a multi-table association query and matching the data rows of the table to be updated, if the data row of the table to be updated matches the result set and meets the WHERE condition, new values are assigned to the specified columns of this data row.

```SQL
[ WITH <with_query> [, ...] ]
UPDATE <table_name>
    SET <column_name> = <expression> [, ...]
    [ FROM <from_item> [, ...] ]
    WHERE <where_condition>
```

## Parameter Description

`with_query`

One or more CTEs that can be referenced by name in the UPDATE statement. CTE is a temporary result set that can improve the readability of complex statements.

`table_name`

The name of the table to be updated.

`column_name`

The name of the column to be updated. It is not necessary to include the table name, for example, `UPDATE t1 SET t1.col = 1` is not valid.

`expression`

The expression that assigns a value to the column.

`from_item`

References one or more other tables in the database. This table is joined with the table to be updated, and the WHERE clause specifies the join conditions. Finally, based on the result set of the join query, the column of the matching rows in the table to be updated is assigned a value. For example, if the FROM clause is `FROM t1 WHERE t0.pk = t1.pk;`, when StarRocks actually executes the UPDATE statement, the table expression in this FROM clause will be converted to `t0 JOIN t1 ON t0.pk=t1.pk;`.

`where_condition`

Only rows that satisfy the WHERE condition will be updated. This parameter is required to prevent accidentally updating the entire table. If you need to update the entire table, please use `WHERE true`. If using [partial updates of column mode](#partial-column-mode-updates-from-31), this parameter is not required.

## Partial Column Mode Updates (from 3.1)

Partial column mode updates are suitable for scenarios involving a small number of columns and a large number of rows. In this scenario, by enabling column mode, the update speed is faster. For example, in a table with 100 columns, if 10 columns (10% proportion) are updated each time and all rows are updated, enabling column mode will improve the performance by 10 times.

The system variable `partial_update_mode` is used to control the mode of partial updates, supporting the following values:

* `auto` (default value), indicating that the system automatically determines the mode used for partial updates by analyzing the update statement and the columns involved. If the following criteria are met, the system automatically uses the column mode:
  * The percentage of the number of columns being updated to the total number of columns is less than 30%, and the number of columns being updated is less than 4.
  * The WHERE condition is not used in the update statement.
  Otherwise, the system will not use the column mode.
* `column`, specifying to use the column mode to perform partial updates, which is more suitable for partial column updates involving a small number of columns and a large number of rows.

You can use `EXPLAIN UPDATE xxx` to view the mode of performing partial column updates.

## Examples

### Single-table UPDATE

Create the `Employees` table to record employee information and insert five rows of data into the table.

```SQL
CREATE TABLE Employees (
    EmployeeID INT,
    Name VARCHAR(50),
    Salary DECIMAL(10, 2)
)
PRIMARY KEY (EmployeeID) 
DISTRIBUTED BY HASH (EmployeeID)
PROPERTIES ("replication_num" = "3");

INSERT INTO Employees VALUES
    (1, 'John Doe', 5000),
    (2, 'Jane Smith', 6000),
    (3, 'Robert Johnson', 5500),
    (4, 'Emily Williams', 4500),
    (5, 'Michael Brown', 7000);
```

If you want to give a 10% raise to all employees, you can execute the following statement:

```SQL
UPDATE Employees
SET Salary = Salary * 1.1  -- Increase salary by 10%
WHERE true;
```

If you want to give a 10% raise to employees with a salary lower than the average salary, you can execute the following statement:

```SQL
UPDATE Employees
SET Salary = Salary * 1.1  -- Increase salary by 10%
WHERE Salary < (SELECT AVG(Salary) FROM Employees);
```

You can also rewrite the above statement using CTE to increase readability.

```SQL
WITH AvgSalary AS (
    SELECT AVG(Salary) AS AverageSalary
    FROM Employees
)
UPDATE Employees
SET Salary = Salary * 1.1  -- Increase salary by 10%
FROM AvgSalary
WHERE Employees.Salary < AvgSalary.AverageSalary;
```

### Multi-table UPDATE

Create the `Accounts` table to record account information and insert three rows of data into the table.

```SQL
CREATE TABLE Accounts (
    Accounts_id BIGINT NOT NULL,
    Name VARCHAR(26) NOT NULL,
    Sales_person VARCHAR(50) NOT NULL
) 
PRIMARY KEY (Accounts_id)
DISTRIBUTED BY HASH (Accounts_id)
PROPERTIES ("replication_num" = "3");

INSERT INTO Accounts VALUES
    (1,'Acme Corporation','John Doe'),
    (2,'Acme Corporation','Robert Johnson'),
    (3,'Acme Corporation','Lily Swift');
```

If you need to give a 10% raise to employees in the `Employees` table who manage accounts for the Acme Corporation, you can execute the following statement:

```SQL
UPDATE Employees
SET Salary = Salary * 1.1  -- Increase salary by 10%
FROM Accounts
WHERE Accounts.name = 'Acme Corporation'
    AND Employees.Name = Accounts.Sales_person;
```

You can also rewrite the above statement using CTE to increase readability.

```SQL
WITH Acme_Accounts as (
    SELECT * from Accounts
    WHERE Accounts.name = 'Acme Corporation'
)
UPDATE Employees SET Salary = Salary * 1.1 -- Increase salary by 10%
FROM Acme_Accounts
WHERE Employees.Name = Acme_Accounts.Sales_person;
```