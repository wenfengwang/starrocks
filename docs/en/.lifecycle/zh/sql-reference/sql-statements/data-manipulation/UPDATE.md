---
displayed_sidebar: English
---

# 更新

更新主键表中的行。

StarRocks 从 v2.3 版本开始支持 UPDATE 语句，该版本仅支持单表 UPDATE，不支持公共表表达式（CTE）。从 3.0 版本开始，StarRocks 丰富了语法，以支持多表连接和 CTE。如果您需要将要更新的表与数据库中的其他表进行连接，可以在 FROM 子句或 CTE 中引用这些其他表。从 3.1 版本开始，UPDATE 语句支持列模式的部分更新，适用于列数较少但行数较多的场景，从而实现更快的更新速度。

此命令要求您对想要更新的表具有 UPDATE 权限。

## 使用说明

当执行涉及多个表的 UPDATE 语句时，StarRocks 会将 UPDATE 语句的 FROM 子句中的表达式转换为等效的 JOIN 查询语句。因此，请确保您在 UPDATE 语句的 FROM 子句中指定的表达式支持此转换。例如，UPDATE 语句是 'UPDATE t0 SET v1=t1.v1 FROM t1 WHERE t0.pk = t1.pk;'。FROM 子句中的表达式可以转换为 't0 JOIN t1 ON t0.pk=t1.pk;'。StarRocks 根据 JOIN 查询的结果集来匹配需要更新的数据行。可能会有多个结果集中的行与要更新的表中的某一行匹配。在这种情况下，该行将根据这些多个行中的任意一行的值进行更新。

## 语法

### 单表更新

如果要更新的表的数据行满足 WHERE 条件，则这些数据行的指定列将被赋予新值。

```SQL
[ WITH <with_query> [, ...] ]
UPDATE <table_name>
    SET <column_name> = <expression> [, ...]
    WHERE <where_condition>
```

### 多表更新

多表连接的结果集与要更新的表进行匹配。如果待更新表的数据行与结果集匹配并满足 WHERE 条件，则这些数据行的指定列将被赋予新值。

```SQL
[ WITH <with_query> [, ...] ]
UPDATE <table_name>
    SET <column_name> = <expression> [, ...]
    [ FROM <from_item> [, ...] ]
    WHERE <where_condition>
```

## 参数

`with_query`

可以在 UPDATE 语句中按名称引用的一个或多个 CTE。CTE 是临时结果集，可以提高复杂语句的可读性。

`table_name`

要更新的表的名称。

`column_name`

要更新的列的名称。它不能包含表名。例如，“UPDATE t1 SET col = 1”是无效的。

`expression`

为列赋予新值的表达式。

`from_item`

数据库中的一个或多个其他表。这些表可以根据 WHERE 子句中指定的条件与要更新的表进行连接。结果集中的行的值用于更新要更新的表中匹配行的指定列的值。例如，如果 FROM 子句是 `FROM t1 WHERE t0.pk = t1.pk`，则 StarRocks 在执行 UPDATE 语句时会将 FROM 子句中的表达式转换为 `t0 JOIN t1 ON t0.pk=t1.pk`。

`where_condition`

您要更新行所依据的条件。只有满足 WHERE 条件的行才能被更新。此参数是必需的，因为它有助于防止您意外更新整个表。如果您想更新整个表，可以使用 'WHERE true'。然而，对于[列模式下的部分更新](#partial-updates-in-column-mode-since-v31)，这个参数不是必需的。

## 列模式下的部分更新（自 v3.1 起）

列模式下的部分更新适用于只需要更新少量列但需要更新大量行的场景。在这种情况下，启用列模式可以提供更快的更新速度。例如，在一个包含 100 列的表中，如果所有行只更新 10 列（占总列数的 10%），则列模式的更新速度将快 10 倍。

系统变量 `partial_update_mode` 控制部分更新的模式，支持以下值：

- `auto`（默认）：系统通过分析 UPDATE 语句和涉及的列来自动确定部分更新的模式。如果满足以下条件，系统将自动使用列模式：
  - 更新的列占总列数的比例小于 30%，且更新的列数少于 4 列。
  - UPDATE 语句不使用 WHERE 条件。
  否则，系统不使用列模式。

- `column`：列模式用于部分更新，特别适合涉及少量列和大量行的部分更新。

您可以使用 `EXPLAIN UPDATE xxx` 来查看部分更新的模式。

## 示例

### 单表更新

创建一张 `Employees` 表来记录员工信息，并向表中插入五条数据行。

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

如果您需要给所有员工加薪 10%，可以执行以下语句：

```SQL
UPDATE Employees
SET Salary = Salary * 1.1  -- Increase the salary by 10%.
WHERE true;
```

如果您需要给工资低于平均工资的员工加薪 10%，可以执行以下语句：

```SQL
UPDATE Employees
SET Salary = Salary * 1.1   -- Increase the salary by 10%.
WHERE Salary < (SELECT AVG(Salary) FROM Employees);
```

您还可以使用 CTE 重写上述语句以提高可读性。

```SQL
WITH AvgSalary AS (
    SELECT AVG(Salary) AS AverageSalary
    FROM Employees
)
UPDATE Employees
SET Salary = Salary * 1.1   -- Increase the salary by 10%.
FROM AvgSalary
WHERE Employees.Salary < AvgSalary.AverageSalary;
```

### 多表更新

创建一个 `Accounts` 表来记录账户信息，并向表中插入三条数据行。

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

如果您需要给 Acme Corporation 管理账户的 `Employees` 表中的员工加薪 10%，您可以执行以下语句：

```SQL
UPDATE Employees
SET Salary = Salary * 1.1  -- Increase the salary by 10%.
FROM Accounts
WHERE Accounts.Name = 'Acme Corporation'
    AND Employees.Name = Accounts.Sales_person;
```

您还可以使用 CTE 重写上述语句以提高可读性。

```SQL
WITH Acme_Accounts AS (
    SELECT * FROM Accounts
    WHERE Accounts.Name = 'Acme Corporation'
)
UPDATE Employees SET Salary = Salary * 1.1 -- Increase the salary by 10%.
FROM Acme_Accounts
WHERE Employees.Name = Acme_Accounts.Sales_person;
```